---
layout: blog
title: "Kubernetes v1.36: Advancing Workload-Aware Scheduling"
draft: true
date: 2026-XX-XX
slug: kubernetes-v1-36-advancing-workload-aware-scheduling
author: >
  Maciej Skoczeń (Google),
  Antoni Zawodny (Google),
  TBD
---

Scheduling large AI/ML and batch workloads is a much more complex operation than scheduling a single Pod.
In Kubernetes v1.35, we introduced the first tranche of *workload-aware scheduling* improvements,
featuring the foundational Workload API alongside basic *gang scheduling* support built on a Pod-based framework,
and an *opportunistic batching* feature to efficiently process identical Pods.

Now, Kubernetes v1.36 brings a massive architectural upgrade. We have redesigned the API to cleanly separate concerns:
the Workload API now serves as a static template, while the new PodGroup API describes the runtime object.
To power this, the `kube-scheduler` features a new *PodGroup scheduling cycle* that enables atomic workload processing
and paves the way for future enhancements. We are also rolling out the first iterations of *topology-aware scheduling*
and *workload-aware preemption* to advance the scheduling capabilities for these workloads. Additionally,
we have added *ResourceClaim support for workloads* to unlock *Dynamic Resource Allocation (DRA)* for PodGroups.
Finally, to demonstrate real-world readiness, v1.36 delivers the first phase of integration between the Job controller
and the new API.

## Workload and PodGroup API updates

The Workload API now serves as a static template, while the new PodGroup API describes the runtime object.
Kubernetes v1.36 introduces the Workload and PodGroup APIs as part of the
`scheduling.k8s.io/v1alpha2` {{< glossary_tooltip text="API group" term_id="api-group" >}},
completely replacing the previous `v1alpha1` API version.

In v1.35, pod groups and their runtime states were embedded within the Workload resource.
In the new model, the Workload object is rarely updated (as a template object),
while the PodGroup handles runtime state. This decoupling also improves performance and scalability,
as the PodGroup API allows per-replica sharding of status updates.

Because the Workload API acts merely as a template, the `kube-scheduler`'s logic is streamlined.
The scheduler can directly read the PodGroup which contains all the information required by scheduler,
without needing to watch or parse the Workload object itself.

Here is what the updated configuration looks like. First, you define the Workload object,
which now acts as a static template for your pod groups:

```yaml
apiVersion: scheduling.k8s.io/v1alpha2
kind: Workload
metadata:
  name: training-job-workload
  namespace: some-ns
spec:
  # Pod groups are now defined as templates
  podGroupTemplates:
  - name: workers
    schedulingPolicy:
      gang:
        # The gang is schedulable only if 4 pods can run at once
        minCount: 4
```

Next, workload controllers (such as the Job controller) stamp out runtime PodGroup instances based on those templates.
The PodGroup runtime object holds the actual scheduling policy and references the template based on which it was created.
The PodGroup also has a status that contains conditions mirroring the states of individual Pods,
reflecting the overall scheduling state of the group.

```yaml
apiVersion: scheduling.k8s.io/v1alpha2
kind: PodGroup
metadata:
  name: training-job-workers-pg
  namespace: some-ns
spec:
  # The PodGroup references the Workload template it originated from
  podGroupTemplateRef:
    workload:
      workloadName: training-job-workload
      podGroupTemplateName: workers
  # The actual scheduling policy is placed inside the runtime PodGroup
  schedulingPolicy:
    gang:
      minCount: 4
status:
  # The status contains conditions mirroring individual Pod conditions.
  conditions:
  - type: PodGroupScheduled
    status: "True"
    lastTransitionTime: 2026-04-03T00:00:00Z
```

Finally, to bridge this new architecture with individual Pods, the workloadRef field in the Pod API has been replaced
with the schedulingGroup field. When creating Pods, you link them directly to the runtime PodGroup:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: worker-0
  namespace: some-ns
spec:
  # The workloadRef field has been replaced by schedulingGroup
  schedulingGroup:
    podGroupName: training-job-workers-pg
  ...
```

By keeping the Workload as a static template and elevating PodGroup to a first-class standalone API,
we establish a robust foundation for building advanced workload scheduling capabilities in future Kubernetes releases.

## PodGroup scheduling cycle and gang scheduling

To efficiently manage these workloads, the kube-scheduler now features a dedicated *PodGroup scheduling cycle*.
Instead of evaluating and reserving resources sequentially Pod-by-Pod, which risks scheduling deadlocks,
the scheduler evaluates the group as a unified operation.

When the scheduler pops a PodGroup member from the scheduling queue, regardless of the group's specific policy,
it fetches the rest of the queued Pods for that group, sorts them deterministically,
and executes an atomic scheduling cycle as follows:

1. The scheduler takes a single snapshot of the cluster state to prevent race conditions and ensure consistency
   while evaluating the entire group.

2. Then, it attempts to find valid Node placements for all Pods in the group using a PodGroup scheduling algorithm,
   which levrrages the standard Pod-based filtering and scoring phases.

3. Based on the algorithm's outcome, the scheduling decision is applied atomically for the entire PodGroup.

   * Success: If the placement is found and group constraints are met, the schedulable member Pods
     are moved directly to the binding phase together.

   * Failure: If the group fails to meet its requirements, the entire group is considered unschedulable.
     None of the Pods are bound, and they are returned to the scheduling queue to retry later after a backoff period.

This cycle acts as the foundation for *gang scheduling*. When your workload requires strict *all-or-nothing* placement,
the `gang` policy leverages this cycle to prevent partial deployments that lead to resource wastage and potential deadlocks.

The scheduler still holds the Pods in the `PreEnqueue` until the `minCount` requirement is met, the actual scheduling phase now relies entirely
on the new PodGroup cycle. Specifically, during the algorithm's execution, the scheduler verifies
that the number of schedulable Pods satisfies the `minCount`. If the cluster cannot accommodate the required minimum,
none of the pods are bound. The group fails and waits for sufficient resources to free up.

### Limitations

The first version of the PodGroup scheduling cycle comes with certain limitations:

* For basic *homogeneous* Pod groups (i.e., those where all Pods have identical scheduling requirements
  and lack inter-Pod dependencies like affinity, anti-affinity, or topology spread constraints),
  the algorithm is expected to find a placement if one exists.

* For *heterogeneous* Pod groups, finding a valid placement is not guaranteed.

* For Pod groups with *inter-Pod dependencies*, finding a valid placement is not guaranteed.

In addition to the above, for cases involving *intra-group dependencies*
(e.g., when the schedulability of one Pod depends on another group member via inter-Pod affinity),
this algorithm may fail to find a placement regardless of cluster state due to its deterministic processing order.

## Topology-aware scheduling

TBD

## Workload-aware preemption

TBD

## DRA ResourceClaim support for workloads

TBD

## Integration with the Job controller

TBD

## What's next?

The journey for workload-aware scheduling doesn't stop here. For v1.37 and beyond, the community is actively working on:

* TBD

The priority and implementation order of these focus areas are subject to change. Stay tuned for further updates.

## Getting started

All below workload-aware scheduling improvements are available as Alpha features in v1.36.
To try them out, you must configure the following:

* Prerequisite: Workload and PodGroup API support: Enable the
  [`GenericWorkload`](/docs/reference/command-line-tools-reference/feature-gates/#GenericWorkload)
  feature gate on both the `kube-apiserver` and `kube-scheduler`, and ensure the `scheduling.k8s.io/v1alpha2`
  {{< glossary_tooltip text="API group" term_id="api-group" >}} is enabled.

Once the prerequisite is met, you can enable specific features:

* Gang scheduling: Enable the
  [`GangScheduling`](/docs/reference/command-line-tools-reference/feature-gates/#GangScheduling)
  feature gate on the `kube-scheduler`.
* Topology-aware scheduling: Enable the
  [`TopologyAwareWorkloadScheduling`](/docs/reference/command-line-tools-reference/feature-gates/#TopologyAwareWorkloadScheduling)
  feature gate on the `kube-scheduler`.
* Workload-aware preemption: Enable the
  [`WorkloadAwarePreemption`](/docs/reference/command-line-tools-reference/feature-gates/#WorkloadAwarePreemption)
  feature gate on the `kube-scheduler` (requires `GangScheduling` to also be enabled).
* DRA ResourceClaim support for workloads: Enable the
  [`DRAWorkloadResourceClaims`](/docs/reference/command-line-tools-reference/feature-gates/#DRAWorkloadResourceClaims)
  feature gate on the `kube-apiserver`, `kube-controller-manager`, `kube-scheduler` and `kubelet`.
* Workload API integration with the Job controller: Enable the
  [`EnableWorkloadWithJob`](/docs/reference/command-line-tools-reference/feature-gates/#EnableWorkloadWithJob)
  feature gate on the `kube-apiserver` and `kube-controller-manager`.

We encourage you to try out workload-aware scheduling in your test clusters
and share your experiences to help shape the future of Kubernetes scheduling.
You can send your feedback by:

* Reaching out via [Slack (#workload-aware-scheduling)](https://kubernetes.slack.com/archives/C0AHLJ0EAEL).
* Joining the [SIG Scheduling](https://docs.google.com/document/d/13mwye7nvrmV11q9_Eg77z-1w3X7Q1GTbslpml4J7F3A/edit)
  or [Workload API integration](https://docs.google.com/document/d/1XSPdK4L3zkAFhAZ3hBQJr2k7JX9CpGD7NeQfujM1PT4/edit) meetings.
* Filing a new [issue](https://github.com/kubernetes/enhancements/issues) in the Kubernetes repository.

## Learn more

To dive deeper into the architecture and design of these features, read the KEPs:

* [Workload API and gang scheduling](https://github.com/kubernetes/enhancements/tree/master/keps/sig-scheduling/4671-gang-scheduling)
* [Topology-aware scheduling](https://github.com/kubernetes/enhancements/tree/master/keps/sig-scheduling/5732-topology-aware-workload-scheduling)
* [Workload-aware preemption](https://github.com/kubernetes/enhancements/tree/master/keps/sig-scheduling/5710-workload-aware-preemption)
* [DRA ResourceClaim support for workloads](https://github.com/kubernetes/enhancements/tree/master/keps/sig-scheduling/5729-resourceclaim-support-for-workloads)
* [Workload API support in Job controller](https://github.com/kubernetes/enhancements/tree/master/keps/sig-apps/5547-integrate-workload-with-job)
