---
tags: maintenance
---
<!-- markdownlint-disable MD013 MD036-->

# Jenkins AKS and EKS Kubernetes Upgrade to 1.20

[![hackmd-github-sync-badge](https://hackmd.io/DIOeeOYVTm6pJeh_dJ9X_A/badge)](https://hackmd.io/DIOeeOYVTm6pJeh_dJ9X_A)

## 1.19.13  to 1.20.x

Related to `publicks8` (AKS) and `cik8s` (EKS) cluster

- [x] JIRA main issue: <https://issues.jenkins.io/browse/INFRA-3118>
  - [x] [upgrade AKS [INFRA-3119]](https://issues.jenkins.io/projects/INFRA/issues/INFRA-3119)
  - [x] [upgrade EKS [INFRA-3121]](https://issues.jenkins.io/projects/INFRA/issues/INFRA-3121)
  - [x] [upgrade kubectl [INFRA-3120]](https://issues.jenkins.io/projects/INFRA/issues/INFRA-3120)

- [X] Announces:
  - [X] `status.jenkins.io`: <https://github.com/jenkins-infra/status/pull/99>
  - [x] Mailing list: https://groups.google.com/g/jenkins-infra/c/MxLu5S94yHM
  - [x] community.jenkins.io: https://community.jenkins.io/t/2021-11-25-aks-and-eks-clusters-upgrade-1-19-1-20-infra-3118/858
  - [x] IRC

- [X] Changelog Scrapping:
  - [X] [Main changes](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.20.md#whats-new-major-themes)
      - [x] From [AKS changelog](https://github.com/Azure/AKS/blob/master/CHANGELOG.md#announcements-18),
        > Before k8s 1.20 a bug would allow exec probes to run indefinitely, ignoring any timeoutSeconds configuration value. The previous buggy behavior has been fixed, and timeouts are now enforced. Additionally, this change introduces a new default timeout of 1 second. Please audit all your existing exec probes to make sure that it is appropriate to enforce a 1 second timeout. If not, please provide an explicit `timeoutSeconds` value that is appropriate for each [exec probe](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#configure-probes).

    The previous timeout period was also 1 second from what I can see, but not respected
    
    - [x] Monitor apps to see which of them will need a specific timeout period (~~probably jenkins~~)
    - [X] The official Jenkins helm chart we're using has already [set the default timeout period to 5 seconds](https://github.com/jenkinsci/helm-charts/blob/main/charts/jenkins/values.yaml#L150-L168)
  - Notes:
    - `kubectl debug` gains support for changing container images when copying a pod for debugging, similar to how kubectl set image works. See `kubectl help debug` for more information.
    - [CronJob controller v2 is available through feature gate](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.20.md#cronjob-controller-v2-is-available-through-feature-gate)
    - The GracefulNodeShutdown feature is now in Alpha. This allows kubelet to be aware of node system shutdowns, enabling graceful termination of pods during a system shutdown. This feature can be enabled through [feature gate.](https://kubernetes.io/docs/concepts/architecture/nodes/#graceful-node-shutdown) (Enabled by default in 1.21)

  - [x] [Urgent Upgrade Notes for 1.20](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.20.md#urgent-upgrade-notes)

  - [x] Azure:
    - From [first release changelog mentioning 1.20](https://github.com/Azure/AKS/blob/master/CHANGELOG.md#release-2021-02-01) to [last one available at the time](https://github.com/Azure/AKS/blob/master/CHANGELOG.md#release-2021-11-18)
    - [release-2021-09-02](https://github.com/Azure/AKS/blob/master/CHANGELOG.md#release-2021-09-02)
      > Scale-down mode is now in public preview docs.microsoft.com/en-us/azure/aks/scale-down-mode
    - [release-2021-08-19](https://github.com/Azure/AKS/blob/master/CHANGELOG.md#release-2021-08-19):
      > Stop ability to make changes to the following system labels:

            beta.kubernetes.io/arch
            beta.kubernetes.io/instance-type
            beta.kubernetes.io/os
            failure-domain.beta.kubernetes.io/region
            failure-domain.beta.kubernetes.io/zone
            failure-domain.kubernetes.io/zone
            failure-domain.kubernetes.io/region
            kubernetes.io/arch
            kubernetes.io/hostname
            kubernetes.io/os
            kubernetes.io/role
            kubernetes.io/instance-type
            node.kubernetes.io/instance-type
            topology.kubernetes.io/region
            topology.kubernetes.io/zone
            kubernetes.azure.com/role=agent
            node-role.kubernetes.io/agent
            kubernetes.io/role=agent
            agentpool
            storageprofile
            storagetier
            accelerator
            kubernetes.azure.com/fips_enabled
            kubernetes.azure.com/os-sku
            kubernetes.azure.com/cluster

    - [release-2021-04-22](https://github.com/Azure/AKS/blob/master/CHANGELOG.md#release-2021-04-22)
      > CSI Drivers will become default for Kubernetes versions 1.21+.
    - [release-2021-03-22](https://github.com/Azure/AKS/blob/master/CHANGELOG.md#release-2021-03-22):
      > Previous [pod security policy (preview)](https://docs.microsoft.com/azure/aks/use-pod-security-policies) deprecation was June 30th 2021. To better align with Kubernetes Upstream pod security policy (preview) deprecation will begin with Kubernetes version 1.21, with its removal in version 1.25. As Kubernetes Upstream approaches that milestone, the Kubernetes community will be working to document viable alternatives.
    - If the upgrade fails, [the support of 1.19 in AKS](https://github.com/Azure/AKS/blob/master/CHANGELOG.md#announcement) is available until January 2022.

  - [x] AWS
    - <https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html>
    - <https://docs.aws.amazon.com/eks/latest/userguide/update-cluster.html>
    - <https://docs.aws.amazon.com/eks/latest/userguide/platform-versions.html>
    - 1.20 brings new [default roles and users](https://docs.aws.amazon.com/eks/latest/userguide/default-roles-users.html)
    - Notable change: 
        > The client-go credential plugins can now be passed in the current cluster information via the KUBERNETES_EXEC_INFO environment variable. This enhancement allows Go clients to authenticate using external credential providers, such as a key management system (KMS).
    - [x] <https://github.com/jenkins-infra/aws/pull/44>

  - [x] Docker Helmfile Upgrade (kubectl):
    - [x] <https://github.com/jenkins-infra/docker-helmfile/pull/40>
    - [x] <https://github.com/jenkins-infra/charts/pull/1754>
  
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
    - [x] Starting the upgrade in Azure (publick8s)
      - [x] Control Plane
      - [x] Windows Node Pool
      - [x] High Memory Node Pool
      - [x] Standard Node Pool
    - [x] Starting the upgrade in AWS (cik8s)
      - [x] Control Plane
      - [x] Standard Node Pool
        > `kubeclt drain` on node still in 1.19 with flag `--ignore-daemonset` evicted autoscaler and `aws-eviction` pods on another node
    - [x] Close maintenance windows in status (<https://github.com/jenkins-infra/status/pull/...>)
    - [x] Send closing email to mailing
    - [x] Send closing message in IRC
    - [x] Write post mortem for what went wrong (if needed)

  - [x] Expected downtimes:
      - [x] ldap, due to its PV reallocation
      - [x] apps which need a longer probe timeout delay

### Notes
