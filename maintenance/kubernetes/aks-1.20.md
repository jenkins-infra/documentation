---
tags: maintenance
---
<!-- markdownlint-disable MD013 MD036-->

# Jenkins AKS Kubernetes Upgrade to 1.20

[![hackmd-github-sync-badge](https://hackmd.io/DIOeeOYVTm6pJeh_dJ9X_A/badge)](https://hackmd.io/DIOeeOYVTm6pJeh_dJ9X_A)

## 1.19.x  to 1.20.x

<!-- not cik8s cluster? -->
Related to `publicks8` and `cik8s` cluster

- [ ] JIRA issue: <https://issues.jenkins.io/browse/INFRA-3118>
    - [ ] [upgrade AKS](https://issues.jenkins.io/projects/INFRA/issues/INFRA-3119)
    - [ ] [upgrade EKS](https://issues.jenkins.io/projects/INFRA/issues/INFRA-3121)
    - [ ] [upgrade kubectl](https://issues.jenkins.io/projects/INFRA/issues/INFRA-3119)

- [ ] Announces:
  - [ ] `status.jenkins.io`: <https://github.com/jenkins-infra/status/pull/...>
  - [ ] Mailing list: 
  - [ ] IRC

- [ ] Changelog Scrapping:
  - [ ] [Main changes](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.20.md#whats-new-major-themes)
  - [ ] [Urgent Upgrade Notes for 1.20](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.20.md#urgent-upgrade-notes)

  
  - Azure:
    - If the upgrade fails, [the support of 1.19 in AKS](https://github.com/Azure/AKS/blob/master/CHANGELOG.md#announcement) is available until January 2022.
    

  - AWS
      - <https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html>
     
  - [ ] Docker Helmfile Upgrade (kubectl): <https://github.com/jenkins-infra/docker-helmfile/pull/...>
  
  - [x] Checking for deprecated APIs on Kube resources/helmfiles
  
  ```shell
  $ pluto detect-files -d ./charts
  There were no resources found with known deprecated apiVersions.
  ```

  - Upgrade Procedure:
    - [ ] Merging last PRs on charts or putting it as on-hold
    - [ ] Pagerduty: Notify the on call person
    - [ ] Stopping the k8s management job
    - [ ] Starting the upgrade in Azure (publick8s)
      - [ ] Control Plane
      - [ ] Windows Node Pool
      - [ ] High Memory Node Pool
      - [ ] Standard Node Pool
    - [ ] Starting the upgrade in AWS (cik8s)
      - [ ] Control Plane
      - [ ] Standard Node Pool
    - [ ] Close maintenance windows in status (<https://github.com/jenkins-infra/status/pull/...>)
    - [ ] Send closing email to mailing
    - [ ] Send closing message in IRC
    - [ ] Write post mortem for what went wrong (if needed)

### Notes

