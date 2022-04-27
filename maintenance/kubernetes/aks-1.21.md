---
tags: maintenance
---
<!-- markdownlint-disable MD013 MD036-->

# Jenkins AKS Kubernetes Upgrade to 1.21

[![hackmd-github-sync-badge](https://hackmd.io/DIOeeOYVTm6pJeh_dJ9X_A/badge)](https://hackmd.io/DIOeeOYVTm6pJeh_dJ9X_A)

## 1.20.x  to 1.21.x

Related to `prodpublicks8` and `temp-privatek8s` clusters

- [x] Issue: <https://github.com/jenkins-infra/helpdesk/issues/2866>

- [X] Announces:
  - [X] `status.jenkins.io`: <https://github.com/jenkins-infra/status/pull/99>
  - [x] IRC

- [X] Changelog Scrapping:
  - Noteworthy:
    - https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.21.md#immutable-secrets-and-configmaps
    - https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.21.md#default-container-annotation
    - https://github.com/kubernetes/enhancements/tree/master/keps/sig-apps/2232-suspend-jobs

  - [x] [Urgent Upgrade Notes for 1.21](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.21.md#urgent-upgrade-notes), nothing to report

  - [x] Azure:
    - <https://github.com/Azure/AKS/blob/master/CHANGELOG.md>: nothing to report, mostly concerns Azure CSI

  - [x] Docker Helmfile Upgrade (kubectl):
    - [x] <hhttps://github.com/jenkins-infra/docker-helmfile/pull/91>
  
  - [x] Checking for deprecated APIs on Kube resources/helmfiles
  
  ```shell
  $ pluto detect-files -d ./charts
  There were no resources found with known deprecated apiVersions.
  ```

  - Upgrade Procedure:
    - [x] Ensure we have access to all clusters in kube-config
    - [x] Docker Helmfile Upgrade (kubectl)
    - [x] Merging last PRs on charts or putting it as on-hold
    - [x] Pagerduty: Notify the on call person
    - [x] Stopping the k8s management job
    - [x] Starting the upgrade in Azure (temp-prodk8s)
    - [x] Starting the upgrade in Azure (prodpublick8s)
      - [x] Control Plane
      - [x] Windows Node Pool
      - [x] High Memory Node Pool
      - [x] Standard Node Pool
    - [x] Close maintenance windows in status (<https://github.com/jenkins-infra/status/pull/...>)
    - [x] Write post mortem for what went wrong (if needed)

  - [x] Expected downtimes: none

### Notes
