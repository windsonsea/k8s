---
layout: blog
title: "PSI Metrics for Kubernetes Graduates to GA"
date: 2026-XX-XX
draft: true
slug: psi-metrics-ga
author: "Maria Fernanda Romano Silva (Google Cloud)"
---

Since its original implementation in the Linux kernel in 2018, Pressure Stall Information (PSI) has provided users with the high-fidelity signals needed to identify resource saturation before it becomes an outage. Unlike traditional utilization metrics, PSI tells the story of tasks stalled and time lost, all in nicely-packaged percentages of time across the CPU, memory, and I/O.

Today, we are excited to announce that Kubelet-integrated PSI metrics have graduated to **General Availability (GA)** in Kubernetes v1.36. This graduation ensures that users across the ecosystem have a stable, reliable interface to observe resource contention at the node, pod, and container levels.

### Beyond Utilization: Why PSI?
Monitoring CPU or memory usage alone can be misleading. A node may report XX% (below 100%) CPU utilization while certain tasks are experiencing severe latency due to scheduling delays. PSI fills this gap by providing:
* **Cumulative Totals**: Absolute time spent in a stalled state.
* **Moving Averages**: 10s, 60s, and 300s windows that allow operators to distinguish between transient spikes and sustained resource tension.

### Proving Stability: Performance Testing at Scale
A common concern when graduating telemetry features is the resource overhead required to collect and serve the metrics. To address this, SIG Node conducted extensive performance validation on high-density workloads (80+ pods) across various machine types.

Our testing focused on four primary scenarios to isolate the impact of the kernel-level trackers and the Kubelet collection logic:
1. **Kernel PSI OFF / Kubelet Feature ON** (Baseline)
2. **Kernel PSI ON / Kubelet Feature ON** (Kernel Scheduler overhead)
3. **Kernel PSI ON / Kubelet Feature OFF** (Default Baseline)
4. **Kernel PSI ON / Kubelet Feature ON** (Feature fallback behavior)

{{< figure src="/images/node_sys_cpu_usage_rate_comparison.png" alt="A line graph comparing the Node System (Kernel) CPU usage rate over elapsed time with the PSI feature turned OFF versus ON." title="Node System CPU Usage Rate Comparison" >}}
*Figure 1: Node System CPU comparison under load (80 pods).*

As seen in Figure 1, the "Kernel Tax" for enabling PSI is remarkably low. Even under heavy I/O and CPU load, the **System CPU** delta between the PSI-enabled (red) and PSI-disabled (blue) clusters remained consistently under **0.2 cores** and over **0.037 cores** for the most part. This confirms that simply enabling the feature does not raise the pre-existing resource use and that the internal kernel bookkeeping for stall tracking is safe for production-scale deployments.

{{< figure src="/images/kubelet_cpu_usage_rate_comprison.png" alt="A line graph comparing the Kubelet CPU usage rate over elapsed time with the PSI feature turned OFF versus ON." title="Kubelet CPU Usage Rate Comparison" >}}
*Figure 2: Kubelet CPU Usage Rate Comparison.*

The Kubelet serves as the primary collector for these metrics. Our results show that while the Kubelet performs periodic "sweeps" to aggregate data from the cgroup hierarchy, the CPU usage rate remains stable. The synchronized bursts seen in Figure 2 are virtually identical in both magnitude and frequency, thus attributable to standard Kubelet housekeeping cycles.

You can read more about our performance tests [here](https://docs.google.com/document/d/1ffv7pleid3uk0dT9euK5vkso41i9tscNoEVgeshfiCc/edit?usp=sharing&resourcekey=0-_xKPCU4zqiGU0e7Q8jDJRw).

### Getting Started
As of v1.36, the `KubeletPSI` feature gate is enabled by default. You can query the Kubelet Summary API to see real-time pressure data:
```
kubectl get --raw "/api/v1/nodes/$(kubectl get nodes -o jsonpath='{.items[0].metadata.name}')/proxy/stats/summary" | jq '.pods[].containers[] | select(.name=="<CONTAINER_NAME>") | {name, cpu: .cpu.psi, memory: .memory.psi, io: .io.psi}'
```

### To Learn More
If you want to dive deeper into how these metrics are calculated and exposed, check out these resources:
1. [The official Kernel documentation](https://docs.kernel.org/accounting/psi.html)
2. [Understanding PSI Kubernetes documentation](https://kubernetes.io/docs/reference/instrumentation/understand-psi-metrics/)
3. [cAdvisor Metrics Implementation](https://github.com/google/cadvisor/blob/master/metrics/prometheus.go) 

### Acknowledgements

This feature was developed through the collaborative efforts of [SIG Node](https://github.com/kubernetes/community/tree/master/sig-node). Special thanks to all contributors who helped design, implement, test, review, and document this feature across its journey from alpha in v1.33, through beta in v1.34, to GA in v1.36.

To provide feedback on this feature, join the [Kubernetes Node Special Interest Group](https://github.com/kubernetes/community/tree/master/sig-node), participate in discussions on the [public Slack channel](http://slack.k8s.io/) (#sig-node), or file an issue on [GitHub](https://github.com/kubernetes/kubernetes/issues).

## Feedback

If you have feedback and want to share your experience using this feature, join the discussion:

- [SIG Node community page](https://github.com/kubernetes/community/tree/master/sig-node)
- [Kubernetes Slack](http://slack.k8s.io/) in the #sig-node channel
- [SIG Node mailing list](https://groups.google.com/forum/#!forum/kubernetes-sig-node)

SIG Node would love to hear about your experiences using this feature in production!