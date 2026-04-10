---  
layout: blog
title: 'Kubernetes v1.36: RELEASE NAME'
date: 2026-04-XXXX
draft: true
evergreen: true
slug: kubernetes-v1-36-release
author: >
  [Kubernetes v1.36 Release Team](https://github.com/kubernetes/sig-release/blob/master/releases/release-1.36/release-team.md)
  release_announcement:
  minor_version: “1.36”
---

**Editors:** Chad M. Crowell,  Kirti Goyal, Sophia Ugochukwu, Swathi Rao, Utkarsh Umre

Similar to previous releases, the release of Kubernetes v1.36 introduces new stable, beta, and alpha features. The consistent delivery of high-quality releases underscores the strength of our development cycle and the vibrant support from our community.

This release consists of 71 enhancements. Of those enhancements, 18 have graduated to Stable, 26 are entering Beta, and 25 have graduated to Alpha.

There are also some [deprecations and removals](#deprecations-and-removals) in this release; make sure to read about those.

## Release theme and logo

\<TODO RELEASE THEME AND LOGO\>

## Spotlight on key updates

Kubernetes v1.36 is packed with new features and improvements. Here are a few select updates the Release Team would like to highlight!

### Stable: Fine-grained API authorization

On behalf of Kubernetes SIG Auth and SIG Node, we are pleased to announce the
graduation of fine-grained `kubelet` API authorization to General Availability
(GA) in Kubernetes v1.36!

The `KubeletFineGrainedAuthz` feature gate was introduced as an opt-in alpha feature in Kubernetes v1.32, then graduated to beta (enabled by default) in v1.33. Now, the feature is generally available, and the feature gate is locked to enable. This feature enables more precise, least-privilege access control over the `kubelet`'s HTTPS API replacing the need to grant the overly broad nodes/proxy permission for common monitoring and observability use cases.

​​This work was done as a part of [KEP #2862](https://github.com/kubernetes/enhancements/issues/2862) led by SIG Auth and SIG Node.

### Beta: Resource Health Status

Previously, Kubernetes lacked a native way to report the health of allocated devices, making it difficult to diagnose Pod crashes caused by hardware failures. Building on the initial Alpha release in v1.31 which focused on Device Plugins, Kubernetes v1.36 expands this feature to support Dynamic Resource Allocation (DRA), introducing the `allocatedResourcesStatus` field to provide a unified health reporting mechanism for all specialized hardware.

Now, users can use `kubectl describe pod` to determine if a container's crash loop is due to an `Unhealthy` or `Unknown` device status, regardless of whether the hardware was provisioned via traditional plugins or the newer DRA framework. This enhanced visibility allows administrators and automated controllers to quickly identify faulty hardware and streamline the recovery of high-performance workloads.
This work was done as part of [KEP #4680](https://github.com/kubernetes/enhancements/issues/4680) led by SIG Node.

### Alpha: Staleness mitigation for controllers

Staleness in Kubernetes controllers is a problem that affects many controllers and can subtly affect controller behavior. It is usually not until it is too late, when a controller in production has already taken incorrect action, that staleness is found to be an issue due to some underlying assumption made by the controller author. Some issues caused by staleness include controllers taking incorrect actions, not taking action when they should, and taking too long to act. We are excited to announce that Kubernetes v1.36 includes new features that help mitigate controller staleness and provide better observability of controller behavior.

This work was done as part of [KEP #5647](https://github.com/kubernetes/enhancements/issues/5647) led by SIG API Machinery.

## Features graduating to Stable

This is a selection of some of the improvements that are now stable following the v1.36 release.

### Volume Group Snapshots

After several cycles in beta, Volume Group Snapshots reach General Availability (GA) in Kubernetes v1.36. This feature allows you to take crash-consistent snapshots across multiple PersistentVolumeClaims simultaneously. By ensuring that data and logs across different volumes remain synchronized, this enhancement provides a robust solution for protecting complex, multi-volume workloads. With this release, the API version promotes to v1 and the CSIVolumeGroupSnapshot feature gate is now locked to enabled.

This work was done as part of [KEP #3476](https://github.com/kubernetes/enhancements/issues/3476) led by SIG Storage.

### CSI driver opt-in for service account tokens via secrets field

### Mutable CSINode Allocatable Property

### Speed up recursive SELinux label change

### API for external signing of Service Account tokens

### DRA features graduating to Stable

### Mutating Admission Policies

### Declarative Validation Of Kubernetes Native Types With validation-gen

### Remove gogo protobuf dependency for Kubernetes API types

### Node log query

### Support User Namespaces in pods

### Support PSI based on cgroupv2

### VolumeSource: OCI Artifact and/or Image


## New features in Beta

This is a selection of some of the improvements that are now beta following the v1.36 release.

### IP/CIDR validation improvements

### Separate kubectl user preferences from cluster configs

### Mutable Container Resources when Job is suspended

### Constrained Impersonation

### DRA features in beta

### Statusz for Kubernetes Components 

### Flagz for Kubernetes Components 

### Mixed Version Proxy (aka Unknown Version Interoperability Proxy)

### Support memory qos with cgroups v2



## New features in Alpha

This is a selection of some of the improvements that are now alpha following the v1.36 release.

### Workload Aware Scheduling (WAS) features in Alpha

### Support scaling to/from zero pods for object/external metrics

### DRA features in Alpha

### Native Histogram Support for Kubernetes Metrics

### Manifest Based Admission Control Config

### Stale Controller Mitigation

### CRI List Streaming



## Graduations, deprecations, and removals in v1.36

### Graduations to stable

This lists all the features that graduated to stable (also known as general availability). For a full list of updates including new features and graduations from alpha to beta, see the release notes.

This release includes a total of XXX enhancements promoted to stable:

- [Support User Namespaces in pods](https://github.com/kubernetes/enhancements/issues/127)
- [API for external signing of Service Account tokens](https://github.com/kubernetes/enhancements/issues/740)
- [Speed up recursive SELinux label change](https://github.com/kubernetes/enhancements/issues/1710)
- [Portworx file in-tree to CSI driver migration](https://github.com/kubernetes/enhancements/issues/2589)
- [Fine grained Kubelet API authorization](https://github.com/kubernetes/enhancements/issues/2862)
- [Mutating Admission Policies](https://github.com/kubernetes/enhancements/issues/3962)
- [Node log query](https://github.com/kubernetes/enhancements/issues/2258)
- [VolumeGroupSnapshot](https://github.com/kubernetes/enhancements/issues/3476)
- [Mutable CSINode Allocatable Property](https://github.com/kubernetes/enhancements/issues/4876)
- [DRA: Prioritized Alternatives in Device Requests](https://github.com/kubernetes/enhancements/issues/4816)
- [Support PSI based on cgroupv2](https://github.com/kubernetes/enhancements/issues/4205)
- [add ProcMount option](https://github.com/kubernetes/enhancements/issues/4265)
- [DRA: Extend PodResources to include resources from Dynamic Resource Allocation](https://github.com/kubernetes/enhancements/issues/3695)
- [VolumeSource: OCI Artifact and/or Image](https://github.com/kubernetes/enhancements/issues/4639)
- [Split L3 Cache Topology Awareness in CPU Manager](https://github.com/kubernetes/enhancements/issues/5109)
- [DRA: AdminAccess for ResourceClaims and ResourceClaimTemplates](https://github.com/kubernetes/enhancements/issues/5018)
- [Remove gogo protobuf dependency for Kubernetes API types](https://github.com/kubernetes/enhancements/issues/5589)
- [CSI driver opt-in for service account tokens via secrets field](https://github.com/kubernetes/enhancements/issues/5538)

## Deprecations removals, and community updates

As Kubernetes develops and matures, features may be deprecated, removed, or replaced with better ones to improve the project's overall health. See the Kubernetes deprecation and removal policy for more details on this process. Kubernetes v1.36 includes a couple of deprecations.

### Ingress NGINX retirement

To prioritize the safety and security of the ecosystem, Kubernetes SIG Network and the Security Response Committee have retired Ingress NGINX on March 24, 2026.
Since that date, there have been no further releases, no bugfixes, and no updates to resolve any security vulnerabilities discovered. Existing deployments of
Ingress NGINX will continue to function, and installation artifacts like Helm charts and container images will remain available. 

For full details, see the [official retirement announcement](/blog/2025/11/11/ingress-nginx-retirement/).

### Deprecate service.spec.externalIPs (#5707)

The `externalIPs` field in Service `spec` is being deprecated, which means you’ll soon lose a quick way to route arbitrary externalIPs to your Services. This
field has been a known security headache for years, enabling man-in-the-middle attacks on your cluster traffic, as documented in [CVE-2020-8554](https://github.com/kubernetes/kubernetes/issues/970760). From Kubernetes v1.36 and onwards, you will see deprecation warnings when using it, with full removal
planned for v1.43.

If your Services still lean on `externalIPs`, consider using LoadBalancer services for cloud-managed ingress, NodePort for simple port exposure, or Gateway API
for a more flexible and secure way to handle external traffic.

For more details on this enhancement, refer to [KEP-5707: Deprecate service.spec.externalIPs](https://kep.k8s.io/5707)

### Remove gitRepo volume driver (#5040)

The gitRepo volume type has been deprecated since v1.11. Starting Kubernetes v1.36, the `gitRepo` volume plugin is permanently disabled and cannot be turned back
on. This change protects clusters from a critical security issue where using `gitRepo` could let an attacker run code as root on the node. 

Although `gitRepo` has been deprecated for years and better alternatives have been recommended, it was still technically possible to use it in previous releases.
From v1.36 onward, that path is closed for good, so any existing workloads depending on `gitRepo` will need to migrate to supported approaches such as init
containers or external git-sync style tools.

For more details on this enhancement, refer to [KEP-5040: Remove gitRepo volume driver](https://kep.k8s.io/5040)

## Release notes

Check out the full details of the Kubernetes v1.36 release in our [release notes](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.36.md).

## Availability

Kubernetes v1.36 is available for download on [GitHub](https://github.com/kubernetes/kubernetes/releases/tag/v1.36.0) or on the [Kubernetes download page](http://releases/download/).

To get started with Kubernetes, check out these interactive tutorials or run local Kubernetes clusters using [minikube](https://minikube.sigs.k8s.io/). You can also easily install v1.34 using kubeadm.

## Release Team

Kubernetes is only possible with the support, commitment, and hard work of its community. Each release team is made up of dedicated community volunteers who work together to build the many pieces that make up the Kubernetes releases you rely on. This requires the specialized skills of people from all corners of our community, from the code itself to its documentation and project management.

We would like to thank the entire [Release Team](https://github.com/kubernetes/sig-release/blob/master/releases/release-1.36/release-team.md) for the hours spent hard at work to deliver the Kubernetes v1.36 release to our community. The Release Team's membership ranges from first-time shadows to returning team leads with experience forged over several release cycles. A very special thanks goes out to our release lead, Ryota Sawada, for guiding us through a successful release cycle, for his hands-on approach to solving challenges, and for bringing the energy and care that drives our community forward.

## Project Velocity

The CNCF K8s [DevStats](https://k8s.devstats.cncf.io/d/11/companies-contributing-in-repository-groups?orgId=1&var-period=m&var-repogroup_name=All) project aggregates a number of interesting data points related to the velocity of Kubernetes and various sub-projects. This includes everything from individual contributions to the number of companies that are contributing, and is an illustration of the depth and breadth of effort that goes into evolving this ecosystem.

During the v1.36 release cycle, which spanned 15 weeks from 12th January 2026 to 22nd April 2026, Kubernetes received contributions from as many as 106 different companies and 491 individuals. In the wider cloud native ecosystem, the figure goes up to 370 companies, counting 2235 total contributors.

Note that “contribution” counts when someone makes a commit, code review, comment, creates an issue or PR, reviews a PR (including blogs and documentation) or comments on issues and PRs.
If you are interested in contributing, visit [Getting Started](https://www.kubernetes.dev/docs/guide/#getting-started) on our contributor website.

Source for this data:
- [Companies contributing to Kubernetes](https://k8s.devstats.cncf.io/d/11/companies-contributing-in-repository-groups?orgId=1&from=1747609200000&to=1756335599000&var-period=d28&var-repogroup_name=Kubernetes&var-repo_name=kubernetes%2Fkubernetes)
- [Overall ecosystem contributions](https://k8s.devstats.cncf.io/d/11/companies-contributing-in-repository-groups?orgId=1&from=1747609200000&to=1756335599000&var-period=d28&var-repogroup_name=All&var-repo_name=kubernetes%2Fkubernetes)


## Events Update

Explore upcoming Kubernetes and cloud native events, including KubeCon + CloudNativeCon, KCD, and other notable conferences worldwide. Stay informed and get involved with the Kubernetes community!

**April 2026**
- KCD - [Kubernetes Community Days: Guadalajara](https://community.cncf.io/events/details/cncf-kcd-guadalajara-presents-kcd-guadalajara-2026/cohost-kcd-guadalajara/): April 18, 2026 | Guadalajara, Mexico

**May 2026**
- KCD - [Kubernetes Community Days: Toronto](https://community.cncf.io/events/details/cncf-kcd-toronto-presents-kcd-toronto-2026/): May 13, 2026 | Toronto, Canada
- KCD - [Kubernetes Community Days: Texas](https://community.cncf.io/events/details/cncf-kcd-texas-presents-kcd-texas-2026/cohost-kcd-texas/): May 15, 2026 | Austin, USA
- KCD - [Kubernetes Community Days: Istanbul](https://community.cncf.io/events/details/cncf-kcd-istanbul-presents-kcd-istanbul-2026/): May 15, 2026 | Istanbul, Turkey
- KCD - [Kubernetes Community Days: Helsinki](https://community.cncf.io/events/details/cncf-kcd-helsinki-presents-kubernetes-community-days-helsinki-2026/): May 20, 2026 | Helsinki, Finland
- KCD - [Kubernetes Community Days: Czech & Slovak](https://community.cncf.io/events/details/cncf-kcd-czech-slovak-presents-kcd-czech-amp-slovak-prague-2026/): May 21, 2026 | Prague, Czechia
 
**June 2026**
- KCD - [Kubernetes Community Days: New York](https://community.cncf.io/events/details/cncf-kcd-new-york-presents-kcd-new-york-2026/): June 10, 2025 | New York, USA

**October 2026**
- KCD - [Kubernetes Community Days: UK: Oct 19, 2026](https://community.cncf.io/events/details/cncf-kcd-uk-presents-kubernetes-community-days-uk-edinburgh-2026/) | Edinburgh, UK

**November 2026**
- KCD - [Kubernetes Community Days: Porto](https://community.cncf.io/events/details/cncf-kcd-porto-presents-kcd-porto-2026-collab-with-devops-days-portugal/): Nov 19, 2026 | Porto, Portugal
- [KubeCon + CloudNativeCon North America 2026](https://events.linuxfoundation.org/kubecon-cloudnativecon-north-america/): Nov 9-12, 2026 | Salt Lake, USA

You can find the latest event details [here](https://community.cncf.io/events/#/list).

## Upcoming Release Webinar

Join members of the Kubernetes v1.36 Release Team on 20 May 2026 to learn about the release highlights of this release. For more information and registration, visit the event page on the CNCF Online Programs site.
Get Involved

The simplest way to get involved with Kubernetes is by joining one of the many [Special Interest Groups](https://github.com/kubernetes/community/blob/master/sig-list.md) (SIGs) that align with your interests. Have something you’d like to broadcast to the Kubernetes community? Share your voice at our weekly [community meeting](https://github.com/kubernetes/community/tree/master/communication), and through the channels below. Thank you for your continued feedback and support.

- Follow us on Bluesky [@Kubernetesio](https://bsky.app/profile/kubernetes.io) for the latest updates
- Join the community discussion on [Discuss](https://discuss.kubernetes.io/)
- Join the community on [Slack](http://slack.k8s.io/)
- Post questions (or answer questions) on [Stack Overflow](http://stackoverflow.com/questions/tagged/kubernetes)
- Share your Kubernetes [story](https://docs.google.com/a/linuxfoundation.org/forms/d/e/1FAIpQLScuI7Ye3VQHQTwBASrgkjQDSS5TP0g3AXfFhwSM9YpHgxRKFA/viewform)
- Read more about what’s happening with Kubernetes on the [blog](https://kubernetes.io/blog/)
- Learn more about the [Kubernetes Release Team](https://github.com/kubernetes/sig-release/tree/master/release-team)