---
tags: maintenance
---
<!-- markdownlint-disable MD013 MD036-->

# Jenkins AKS Kubernetes Upgrade to 1.19

[![hackmd-github-sync-badge](https://hackmd.io/EDpvZx9ZS2GWgHHqWFRDfQ/badge)](https://hackmd.io/EDpvZx9ZS2GWgHHqWFRDfQ)

## 1.18.x  to 1.19.x

Related to `publicks8` cluster

- JIRA issue: <https://issues.jenkins.io/browse/INFRA-3005>

- Announces:
  - `status.jenkins.io`: <https://github.com/jenkins-infra/status/pull/38> :heavy_check_mark:
  - Mailing list: <https://groups.google.com/g/jenkins-infra/c/SlrgijwLa5A> :heavy_check_mark:
  - IRC :heavy_check_mark:

- Changelog Scrapping:
  - [Main changes](https://github.com/kubernetes/kubernetes/blob/release-1.19/CHANGELOG/CHANGELOG-1.19.md#whats-new-major-themes) :heavy_check_mark:
  - [Urgent Upgrade Notes for 1.19](https://github.com/kubernetes/kubernetes/blob/release-1.19/CHANGELOG/CHANGELOG-1.19.md#urgent-upgrade-notes) mentions a deprecation around [Azure Blob Storage](https://github.com/kubernetes/kubernetes/blob/release-1.19/CHANGELOG/CHANGELOG-1.19.md#urgent-upgrade-notes) - Status: :heavy_check_mark:

  ```text
  Azure blob disk feature(kind: Shared, Dedicated) has been deprecated, 
  you should use kind: Managed in kubernetes.io/azure-disk storage class.
  (#92905, @andyzhangx) [SIG Cloud Provider and Storage]
  ```
  
  ```shell
  $ kubectl get storageclasses.storage.k8s.io -o yaml \
    | yq eval '.items[] | .metadata.name + ": " + .parameters.kind' -
  default: Managed
  managed: Managed
  managed-premium: Managed
  ```
  
  - Azure:
    - If the upgrade fails, [the support of 1.18 in AKS](https://github.com/Azure/AKS/blob/master/CHANGELOG.md#release-2021-06-10) is extended until end of July 2021. But better to thing upgrading to 1.20 in July...
    - Containerd is the default in 1.19 (instead of Docker). More on the limitations: <https://docs.microsoft.com/en-us/azure/aks/cluster-configuration#containerd-limitationsdifferences>
      - No more `/var/run/docker.sock` - To be checked
        - No reference to `docker.sock` in the charts repository
        - References found in [datadog related resources in jenkins-infra/jenkins-infra](https://github.com/jenkins-infra/jenkins-infra/blob/4f1efa936df8bcfe7a2404c4763fc9e4eda7d8d0/dist/profile/templates/kubernetes/resources/datadog/)
        - Checking the actual state of the cluster:

        ```shell
        ➜  kubectl get daemonsets.apps --all-namespaces -o yaml | grep 'docker.sock'
            value: unix:///host/var/run/docker.sock
            value: unix:///host/var/run/docker.sock
            value: unix:///host/var/run/docker.sock
            value: unix:///host/var/run/docker.sock
          - mountPath: /host/var/run/docker.sock
            name: docker-socket
            path: /var/run/docker.sock
          name: docker-socket
        ➜  charts git:(0ab724e) ✗ kubectl get deployments.apps --all-namespaces -o yaml | grep 'docker.sock' 
        ```

        => Culprits are falco and datadog Daemonset. Currentl searching at solutions for them.

        - Datadog: Support since (..) .<https://github.com/helm/charts/tree/master/stable/datadog#cri-integration> - <https://github.com/DataDog/datadog-agent/issues/3701> :heavy_check_mark: (<https://github.com/jenkins-infra/charts/pull/1258>)

        - Falco: Support since [v0.15.0](https://github.com/falcosecurity/falco/blob/master/CHANGELOG.md#v0150). Containerd socket is already mounted in the daemonset so nothing to do here :heavy_check_mark:

      - Logging format changing. No impact expected: loki helm chart manages. :heavy_check_mark:

  - Docker Helmfile Upgrade (kubectl): <https://github.com/jenkins-infra/docker-helmfile/pull/25>
  
  - Checking for deprecated APIs on Kube resources/helmfiles :heavy_check_mark:
  
  ```shell
  $ pluto detect-files -d ./charts
  There were no resources found with known deprecated apiVersions.
  ```

  - Upgrade Procedure:
    - Merging last PRs (including datadog as last one) on charts or putting it as on-hold :heavy_check_mark:
    - Pagerduty: Notify the on call person: :heavy_check_mark:
    - Stopping the k8s management job: :heavy_check_mark:
    - Starting the upgrade in Azure: :heavy_check_mark:
      - Control Plane :heavy_check_mark:
      - Windows Node Pool :heavy_check_mark:
      - High Memory Node Pool :heavy_check_mark:
      - Standard Node Pool :warning: (see notes below)
    - Close maintenance windows in status (<https://github.com/jenkins-infra/status/pull/39>) :heavy_check_mark:
    - Send closing email to mailing :heavy_check_mark:
    - Send closing message in IRC :heavy_check_mark:
    - Write post mortem for what went wrong :heavy_check_mark:

### Notes

**Node Upgrade Failures - SNAT ports exhaustion - PVC mount timeout**

The migration was **almost** as success. But it failed for the 2 last nodes of the pool "agent node".

Azure UI and CLI reported that the migration was failed, but the kubelet where upgraded on the 2 machines. However these machines were not upgraded at the OS level, and kept the pods that should have been drained.

The root cause was found to be SNAT port exhaustion (Azure UI -> Kubernetes -> publick8s -> Diagnose and solve problems -> Cluster Insights).

An operation was made by the team a few weeks ago (ping @olblak @zvW_c6ROSOOuJDTOracA7Q did we track this somewhere?) to add more public IPs to increase the threshold as per <https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-outbound-connections#snat>, but the pools of public IP was not associated to the AKS cluster's loadbalancer anymore, the reason is not known.

Associating again the pool of public IPs, and adding it to the public LB outbound rule solved all the PVC mount timeout issues.

**ldpa.jenkins.io outage**

- Post Mortem: <https://hackmd.io/8H_OxGydSRiiVGE9VpFcVw#>
- 6 minutes of outage due to the PVC mount timeout described earlier

**Datadog Agents failing to dial with Kubelete**

While checking for the containerd proper usage for datadog-agent, it appeared that the datadog agents were not able to reach the Kubelet anymore since thethe kubelet is only exposed throught HTTP+TLS now.

Fix was quick and easy: <https://github.com/jenkins-infra/charts/pull/1265> (includes documentation references in the PR)
