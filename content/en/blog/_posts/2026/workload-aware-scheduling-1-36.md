---
layout: blog
title: "Kubernetes v1.36: Advancing Workload-Aware Scheduling"
draft: true
date: 2026-XX-XX
slug: kubernetes-v1-36-advancing-workload-aware-scheduling
author: >
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
and *workload-aware preemption* to advance the scheduling capabilities for these workloads.
Finally, to demonstrate real-world readiness, v1.36 delivers the first phase of integration between the Job controller
and the new API.

## Workload and PodGroup API updates

TBD

## PodGroup scheduling cycle and gang scheduling

TBD

## Topology-aware scheduling

TBD

## Workload-aware preemption

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
* [Workload API support in Job controller](https://github.com/kubernetes/enhancements/tree/master/keps/sig-apps/5547-integrate-workload-with-job)
