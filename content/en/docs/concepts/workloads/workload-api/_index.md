---
title: "Workload API"
weight: 20
simple_list: true
---

<!-- overview -->
{{< feature-state feature_gate_name="GenericWorkload" >}}

The `Workload` API resource defines the scheduling requirements and structure of a multi-Pod
application. While workload controllers such as [Job](/docs/concepts/workloads/controllers/job/)
manage the application's runtime state, the `Workload` specifies how groups of `Pods`
should be scheduled. The Job controller is the only built-in controller that creates
[PodGroup](/docs/concepts/workloads/podgroup-api/) objects from the `Workload`'s
`PodGroupTemplates` at runtime.

<!-- body -->

## What is a Workload?

The Workload API resource is part of the `scheduling.k8s.io/v1alpha2`
{{< glossary_tooltip text="API group" term_id="api-group" >}}
and your cluster must have that API group enabled, as well as the `GenericWorkload`
[feature gate](/docs/reference/command-line-tools-reference/feature-gates/),
before you can use this API.

A `Workload` is a static, long-lived policy template. It defines what scheduling
policies should be applied to groups of Pods, but does not track runtime state itself.
Runtime scheduling state is maintained by [PodGroup](/docs/concepts/workloads/podgroup-api/)
objects, which controllers create from the `Workload`'s `PodGroupTemplates`.

## API structure

A `Workload` consists of two fields: a list of `PodGroupTemplates` and an optional controller
reference. The entire `Workload` spec is immutable after creation: you cannot modify
existing templates, add new templates, or remove templates from `podGroupTemplates`.

### PodGroupTemplates

The `spec.podGroupTemplates` list defines the distinct components of your workload.
For example, a machine learning job might have a `driver` template and a `worker` template.

Each entry in `podGroupTemplates` must have:
1. A unique `name` that will be used to reference the template in the `PodGroup`'s `spec.podGroupTemplateRef`.
2. A [scheduling policy](/docs/concepts/workloads/workload-api/policies/) (`basic` or `gang`).

If the [`WorkloadAwarePreemption`](/docs/reference/command-line-tools-reference/feature-gates/#WorkloadAwarePreemption) [feature gate](/docs/reference/command-line-tools-reference/feature-gates/) is enabled each entry in `podGroups` can also have [priority and disruption mode](/docs/concepts/workloads/workload-api/disruption-and-priority/).

The maximum number of PodGroupTemplates in a single Workload is 8.

```yaml
apiVersion: scheduling.k8s.io/v1alpha2
kind: Workload
metadata:
  name: training-job-workload
  namespace: some-ns
spec:
  controllerRef:
    apiGroup: batch
    kind: Job
    name: training-job
  podGroupTemplates:
  - name: workers
    schedulingPolicy:
      gang:
        # The gang is schedulable only if 4 pods can run at once
        minCount: 4
    priorityClassName: high-priority # Only applicable with WorkloadAwarePreemption feature gate
    disruptionMode: PodGroup # Only applicable with WorkloadAwarePreemption feature gate
```

When a workload controller creates a `PodGroup` from one of these templates, it copies the
`schedulingPolicy` into the `PodGroup`'s own spec. Changes to the `Workload` only affect
newly created `PodGroups`, not existing ones.

### Referencing a workload controlling object

The `controllerRef` field links the Workload back to the specific high-level object defining the application,
such as a [Job](/docs/concepts/workloads/controllers/job/) or a custom CRD. This is useful for observability and tooling.
This data is not used to schedule or manage the Workload.

### Requesting DRA devices for a PodGroup

{{< feature-state feature_gate_name="DRAWorkloadResourceClaims" >}}

{{< glossary_tooltip text="Devices" term_id="device" >}} available through
{{< glossary_tooltip text="Dynamic Resource Allocation (DRA)" term_id="dra" >}}
can be requested by a PodGroup through its `spec.resourceClaims` field:

```yaml
apiVersion: scheduling.k8s.io/v1alpha2
kind: PodGroup
metadata:
  name: training-group
  namespace: some-ns
spec:
  ...
  resourceClaims:
  - name: pg-claim
    resourceClaimName: my-pg-claim
  - name: pg-claim-template
    resourceClaimTemplateName: my-pg-template
```

{{< glossary_tooltip text="ResourceClaims" term_id="resourceclaim" >}}
associated with PodGroups can be shared by all Pods belonging to the group. With
only a reference to the PodGroup in the ResourceClaim's `status.reservedFor`
instead of each individual Pod, any number of Pods in the same PodGroup can
share a ResourceClaim. ResourceClaims can also be generated from
{{< glossary_tooltip text="ResourceClaimTemplates" term_id="resourceclaimtemplate" >}}
for each PodGroup, allowing the devices allocated to each generated
ResourceClaim to be shared by the Pods in each PodGroup.

For more details and a more complete example, see the
[DRA documentation](/docs/concepts/scheduling-eviction/dynamic-resource-allocation/#workload-resource-claims).

## Gang scheduling with Jobs

{{< feature-state feature_gate_name="EnableWorkloadWithJob" >}}

When the
[`EnableWorkloadWithJob`](/docs/reference/command-line-tools-reference/feature-gates/)
feature gate is enabled, the
[Job](/docs/concepts/workloads/controllers/job/) controller automatically
creates Workload and PodGroup objects for parallel indexed Jobs where
`.spec.parallelism` equals `.spec.completions`. The gang policy's `minCount`
is set to the Job's parallelism, so all Pods must be schedulable together
before any of them are bound to nodes.

This is the built-in path for using gang scheduling with Jobs.
You do not need to create Workload or PodGroup objects yourself as the Job
controller handles it automatically. Other workload controllers (such as
JobSet) may manage their own Workload and PodGroup objects independently.

## {{% heading "whatsnext" %}}

* Learn about [PodGroup scheduling policies](/docs/concepts/workloads/workload-api/policies/).
* See how PodGroups are created from Workloads in the [PodGroup API](/docs/concepts/workloads/podgroup-api/) overview.
* Read about how Pods reference their PodGroup via the [scheduling group](/docs/concepts/workloads/pods/scheduling-group/) field.
* Learn about [Topology-aware workload scheduling](/docs/concepts/workloads/workload-api/topology-aware-scheduling/).
* Understand the [gang scheduling](/docs/concepts/scheduling-eviction/gang-scheduling/) algorithm.
