---
tags: post-mortem
state: closed
project: infrastructure
---
<!-- markdownlint-disable MD013 -->

# 2021-05-25 - Weekly Release Done Twice + Windows Packaging Unavailable

**! This document is open to feedback until the 1th of June 2021 then this document will be archived on [jenkins-infra/documentation](https://github.com/jenkins-infra/documentation) !**

## Chronology

* 2021-05-25 - 09:30 UTC: The weekly release process for the version 2.294 is triggered as part of the nominal process
* 2021-05-25 - 11:42 UTC: a PagerDuty alert is triggered about the weekly package for Windows not available since more than 30 min
* 2021-05-25 - ~14:00 UTC: the weekly release process is aborted manually, due to the packaging stage being stuck at the Windows packaging step, due to Windows (Kubernetes) Pods not being allocated for the agent

* 2021-05-25 - ~14:27 UTC: in order to validate a fix on the Windows pods allocations, the weekly release build is relaunched accidentally instead of the packaging process, triggering a new release 2.295
* 2021-05-25 - ~16:27 UTC: the 2nd weekly job is aborted for the same reason as the previous one: stuck at packaging Windows, due to pod not allocated (but with a different error)
* 2021-05-26 - 07:35 UTC: the packaging job is executed again manually after fixing a few Kubernetes-configuration items, and succeeded after 14min
* 2021-05-26 - 07:51 UTC: the weekly release 2.295 is fully available online, end of the pager duty errors

## Impacts

* The Windows package for the weekly Jenkins release was unavailable from 2021-05-25 11:30 UTC until 2021-05-26 07:51 UTC.
* The release 2.294 has been duplicated as the 2.295, leading to additional changelog, messages update

## Technical Elements

* Weekly release build for 2.294: <https://release.ci.jenkins.io/job/core/job/weekly/job/release/job/master/63/>
* Weekly release build for 2.295: <https://release.ci.jenkins.io/job/core/job/weekly/job/release/job/master/64/>
* Weekly Core Package build fixing the Windows package: <https://release.ci.jenkins.io/job/core/job/package/job/master/118/>
* Configuration fix for the Jenkins controller instance release.ci.jenkins.io: <https://github.com/jenkins-infra/charts/pull/1196>
* Configuration fixes for the release & package processes:
  * <https://github.com/jenkins-infra/release/commit/2d1f11145a1490e116f323fa7fdd19877e6917bc>
  * <https://github.com/jenkins-infra/release/commit/92dde3d7af5f869a31644a0ea65d27fe0333eacd>

## What Went Wrong?

* The previous pod template configuration for the release & package build (windows - <https://github.com/jenkins-infra/release/blob/master/PodTemplates.d/package-windows>), is mounting the azure file storage volume directly into the pod to ease the deployment of the generating artefacts
  * The `mountPath` attribute was using a nested mount to avoid issues with the Windows UNC path: it as a hack used at the beginning of Windows support for Kubernetes where a 1st `volumeMounts` directive defined the mountpoing with an UNC path (e.g. `mountPath: 'C:\Packages'`), and the other sub mounts could then use Unix-like relative path on a 2nd `volumeMounts` (e.g. `mountPath: /Packages/something`).
  * During the incident, Kubernetes 1.18.4 was used.
  * Since <https://github.com/kubernetes/kubernetes/pull/90162> (introduced in Kubernetes 1.18.3), this "hack" was working, but creating way more requests on the volume
* The release happened while the azure filestorage host was throttled from the API: the additional amount of request made the pod unable to be started because the volume mount was not possible in the expected time (throttled API request)
  * The pod scheduling system was then stuck in a loop (bug fixed in 1.18.5 as per <https://github.com/kubernetes/kubernetes/pull/91307>) because of too frequent retries. The node pool for Windows host was scaled to its upper limit trying to spawn a lot of pods.
  * Being unable to schedule any pods, the package job was stuck, leading to a lot of errors like `[Warning][release/packaging-windows-ksdw2-2vcjb][FailedScheduling] 0/12 nodes are available: 10 Insufficient cpu, 2 node(s) didn't match node selector, 5 Insufficient memory.`
* A set of fixes had been applied to ensure that the Windows pods where using correct UNC paths on all pods templates (both global ones at Jenkins level, and in the jenkins-infra/release repository)
* A human error (from I, @dduportal, the writer of these lines) led to the wrong job being replayed in order to finally publish the Windows package (Release job instead of package job) => It led to a 2nd weekly release done accidentally.
  * Sorry for that :|
* The fixes where correct, but the file storage quota was hit, so we had to wait the next day (2021-05-26) to launch the publication again

## Proposal Improvements

### Short term

* Configuration fix for the Jenkins controller instance release.ci.jenkins.io (setting UNC paths, increasing timeout and adding an upper limit): <https://github.com/jenkins-infra/charts/pull/1196>
* Configuration fixes for the release & package processes (removing duplicated `volumeMount` keys and using UNC paths):
  * <https://github.com/jenkins-infra/release/commit/2d1f11145a1490e116f323fa7fdd19877e6917bc>
  * <https://github.com/jenkins-infra/release/commit/92dde3d7af5f869a31644a0ea65d27fe0333eacd>
* Adding a note on status.jenkins.io: <https://github.com/jenkins-infra/status/pull/18>

### Medium-term

* Questioning (and eventually updating) the package process to NOT mount the Azure filesystem, avoiding the pressure on the scheduling system
  * What was the reason to use a mounted volume instead of the Azure CLI? Are this reason still valid given the impact we saw?
    * @olblak : The motivation is to hide that abstraction at the Kubernetes level. We mount the volume directly into the container so the script that package window artifact and then publish them doesn't need to know if the volume is an Azure or AWS or whatever else we decide to use. The packaging script just has copy artifacts from a directory to another directory in the container. The benefits is we don't have to maintain additional credentials in Jenkins and we don't need to install azure-cli in the window container
* Adding safeguards around the retries to fail after a certain timeout for the packaging step

### Long-term

* Finding a way to avoid a human doing the same mistake (e.g. 2 releases)
  * Team Knowledge Sharing?
  * Improving the (already complete) documentation?
    * @olblak: Only the packaging [doc](https://github.com/jenkins-infra/release#packaging) explains the consequences of re-triggering the job
  * Adding a message on the job description?
    * @olblak: That would be a good idea to add a message [here](https://github.com/jenkins-infra/charts/blob/master/config/default/jenkins-release.yaml#L323)
  * Other ideas?
