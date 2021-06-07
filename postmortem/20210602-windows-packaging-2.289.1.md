---
tags: post-mortem
state: closed
project: infrastructure
---

<!-- markdownlint-disable MD013 -->
# 2021-06-02 - LTS (2.289.1) Windows Packaging Unavailable

**! This document is open to feedback until the 14th of June 2021 then this document will be archived on [jenkins-infra/documentation](https://github.com/jenkins-infra/documentation) !**

## Chronology

* 2021-06-02 - 16:15 UTC: The packaging process for the version LTS 2.289.1 is triggered as part of the nominal process
* 2021-06-02 - 19:00 UTC: a PagerDuty alert is triggered about the stable package for Windows not available
* 2021-06-02 - ~21:00 UTC: a Message in the IRC channel #jenkins-infra confirms that there is an outage on the Windows LTS package
* 2021-06-03 - 05:18 UTC: the LTS package build is aborted manually, as it was stuck since 13h trying to allocate a Windows Kubernetes Pod
* 2021-06-03 - 06:39 UTC: a [fix](https://github.com/jenkins-infra/release/pull/164) is approved and merged
* 2021-06-03 - 06:51 UTC: the packaging job is executed again manually and succeeded after 12min
* 2021-06-03 - 07:03 UTC: the LTS release 2.289.1 Windows pacakage is generally available, end of the pager duty errors

## Impacts

* The Windows package for the LTS Jenkins release was unavailable from 2021-06-02 16:30 UTC until 2021-06-03 07:03 UTC.

## Technical Elements

* Aborted LTS packaging build for 2.289.1: <https://release.ci.jenkins.io/job/core/job/package/job/stable-2.289/2/>
* Succeeded LTS packaging build for 2.289.1: <https://release.ci.jenkins.io/job/core/job/package/job/stable-2.289/4/>
* Configuration fix (see section below) for stable release branch: <https://github.com/jenkins-infra/release/pull/164>
* Configuration fix backported to weekly's branch: <https://github.com/jenkins-infra/release/pull/165>

## What Went Wrong?

* The Windows pod initially allocated during the first build failed to start, because of a persistent volume mount error:

```text
[Warning][release/packaging-windows-k2wtl-wvcx9][Failed] Error: failed to start container "dotnet": Error response from daemon: error while creating mount source path 'c:\var\lib\kubelet\pods\6eadf1bd-e21a-4885-b386-b3a4333014e8\volumes\kubernetes.io~azure-file\binary-core-packages': mkdir c:\var\lib\kubelet\pods\6eadf1bd-e21a-4885-b386-b3a4333014e8\volumes\kubernetes.io~azure-file\binary-core-packages: Cannot create a file when that file already exists.
```

* This error did not happen on the previous build, neither on the previous weekly which has the same configuration

* After further research, it appears that the Azure File storage was under heavy load and the AKS API were throttled at this point on time, resulting in the a failure to finish the pod scheduling: timeout for starting the agent is superior to the throttled API answers for mounting the persistent volume

* Then the same domino effect as last week happened: as the packaging process does not have a circuit breaker, the build on release.ci.jenkins.io retried to spawn a pod every 10 min until the build was manually aborted
  * The 10 min value is a remnant of last week incident. The value was 3 min before, and was increased to avoid causing too much pressure on the azure file storage. It was a success: the node autoscaling was not triggered, no money was wasted on additional VM being started for nothing.

* While investigating the release process, it appears that the "deployment" phase is not executed by the Windows pod agent: it only builds the MSI file and archive it as part of the build, and then the principal Linux pod is used to deploy this archived artifact

* Therefore the fix was easy: removing the volumes and volumeMounts from the Windows pod to reduce the pressure on the AKS scheduler
** The Linux pod is scheduled at the beginning of the build: failure to schedule it would fail fast the job (because no possible clone)

* Once the fix was applied, the Windows pod started in less than 1 min, with only 2 API request (instead of ~20 before)

## Proposal Improvements

### Short term

* Configuration fixes
  * Stable: <https://github.com/jenkins-infra/release/pull/164>
  * Weekly: <https://github.com/jenkins-infra/release/pull/165>

### Medium-term

* Adding safeguards around the retries to fail after a certain timeout for the packaging step

### Long-term

* Improving the release process to stage the packages priori to the "real" deployment day
