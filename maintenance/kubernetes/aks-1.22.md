---
tags: maintenance
---
<!-- markdownlint-disable MD013 MD036-->

# Jenkins AKS Kubernetes Upgrade to 1.22

https://docs.microsoft.com/en-us/azure/aks/upgrade-cluster?tabs=azure-cli

## 1.21.x  to 1.22.x

Related to `prodpublicks8` and `temp-privatek8s` clusters

- [x] Issue: <https://github.com/jenkins-infra/helpdesk/issues/2930>

- [X] Announces:
  - [X] `status.jenkins.io`: <https://github.com/jenkins-infra/status/pull/99>
  - [x] IRC
  - [x] ML (jenkins-infra@googlegroups.com, jenkinsci-dev@googlegroups.com) [template bellow]

- [X] Proceed to the upgrade
  - [X] Changelog Scrapping:
    - [X] [Urgent Upgrade Notes for 1.22](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.22.md#urgent-upgrade-notes), nothing to report
    - [X] [Azure Changelog](https://github.com/Azure/AKS/blob/master/CHANGELOG.md): nothing to report
      - [X] [Selected quotes](https://github.com/jenkins-infra/helpdesk/issues/2930#issuecomment-1168457423)

  - [x] Docker Helmfile Upgrade (kubectl):
    - [x] <https://github.com/jenkins-infra/docker-helmfile/pull/105>
  
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

```console
  # == temp-privatek8s upgrade ==
  # Go to https://infra.ci.jenkins.io/prepareShutdown/

  # login
  $ az login

  # available upgrades
  $ az aks get-upgrades --resource-group prod-jenkins-private-prod --name temp-privatek8s --output table
  Name     ResourceGroup    MasterVersion    Upgrades
  -------  ---------------  ---------------  --------------
  default  temp-privatek8s  1.21.9           1.22.4, 1.22.6

  # upgrade control-plane first
  $ az aks upgrade \
      --resource-group prod-jenkins-private-prod \
      --name temp-privatek8s \
      --control-plane-only \
      --kubernetes-version 1.22

      Kubernetes may be unavailable during cluster upgrades.
      Are you sure you want to perform this operation? (y/N): y
     Since control-plane-only argument is specified, this will upgrade only the control plane to 1.22. Node pool will not change. Continue? (y/N): y

  $ az aks show --resource-group prod-jenkins-private-prod --name temp-privatek8s --output table
  Name             Location    ResourceGroup              KubernetesVersion    CurrentKubernetesVersion    ProvisioningState    Fqdn
  ---------------  ----------  -------------------------  -------------------  --------------------------  -------------------  --------------------------------------------------
  temp-privatek8s  eastus2     prod-jenkins-private-prod  1.22                 1.22.6                      Upgrading            temp-privatek8s-dns-c5f4426a.hcp.eastus2.azmk8s.io

  $ az aks show --resource-group prod-jenkins-private-prod --name temp-privatek8s --output table
  Name             Location    ResourceGroup              KubernetesVersion    CurrentKubernetesVersion    ProvisioningState    Fqdn
  ---------------  ----------  -------------------------  -------------------  --------------------------  -------------------  --------------------------------------------------
  temp-privatek8s  eastus2     prod-jenkins-private-prod  1.22                 1.22.6                      Succeeded            temp-privatek8s-dns-c5f4426a.hcp.eastus2.azmk8s.io

  # upgrade the nodes
  $ az aks upgrade \
      --resource-group prod-jenkins-private-prod \
      --name temp-privatek8s \
      --kubernetes-version 1.22

      Kubernetes may be unavailable during cluster upgrades.
      Are you sure you want to perform this operation? (y/N): y
     The cluster is already on version 1.22 and is not in a failed state. No operations will occur when upgrading to the same version if the cluster is not in a failed state.
     Since control-plane-only argument is not specified, this will upgrade the control plane AND all nodepools to version 1.22. Continue? (y/N): y

  $ az aks show --resource-group prod-jenkins-private-prod --name temp-privatek8s --output table
  Name             Location    ResourceGroup              KubernetesVersion    CurrentKubernetesVersion    ProvisioningState    Fqdn
  ---------------  ----------  -------------------------  -------------------  --------------------------  -------------------  --------------------------------------------------
  temp-privatek8s  eastus2     prod-jenkins-private-prod  1.22                 1.22.6                      Succeeded            temp-privatek8s-dns-c5f4426a.hcp.eastus2.azmk8s.io
```

   - [x] Starting the upgrade in Azure (prodpublick8s)
      
```console
  # == prodpublick8s upgrade ==
  # Go to https://release.ci.jenkins.io/prepareShutdown/

  # TODO: Deactivate "Kubernetes Management" job on infra.ci.jenkins.io


  # available upgrades
  $ az aks get-upgrades --resource-group prodpublick8s --name prodpublick8s --output table
  Name     ResourceGroup    MasterVersion    Upgrades
  -------  ---------------  ---------------  --------------
  default  prodpublick8s    1.21.9           1.22.4, 1.22.6

  # upgrade control-plane first
  $ az aks upgrade \
      --resource-group prodpublick8s \
      --name prodpublick8s \
      --control-plane-only \
      --kubernetes-version 1.22

      Kubernetes may be unavailable during cluster upgrades.
      Are you sure you want to perform this operation? (y/N): y
     Since control-plane-only argument is specified, this will upgrade only the control plane to 1.22. Node pool will not change. Continue? (y/N): y

  $ az aks show --resource-group prodpublick8s --name prodpublick8s --output table
  Name           Location    ResourceGroup    KubernetesVersion    CurrentKubernetesVersion    ProvisioningState    Fqdn
  -------------  ----------  ---------------  -------------------  --------------------------  -------------------  ------------------------------------------------
  prodpublick8s  eastus2     prodpublick8s    1.22                 1.22.6                      Upgrading            prodpublick8s-dns-bdcd7389.hcp.eastus2.azmk8s.io

  $ az aks show --resource-group prodpublick8s --name prodpublick8s --output table
  Name           Location    ResourceGroup    KubernetesVersion    CurrentKubernetesVersion    ProvisioningState    Fqdn
  -------------  ----------  ---------------  -------------------  --------------------------  -------------------  ------------------------------------------------
  prodpublick8s  eastus2     prodpublick8s    1.22                 1.22.6                      Succeeded            prodpublick8s-dns-bdcd7389.hcp.eastus2.azmk8s.io

  # upgrade the nodes
  $ az aks upgrade \
      --resource-group prodpublick8s \
      --name prodpublick8s \
      --kubernetes-version 1.22

      Kubernetes may be unavailable during cluster upgrades.
      Are you sure you want to perform this operation? (y/N): y
     The cluster is already on version 1.22 and is not in a failed state. No operations will occur when upgrading to the same version if the cluster is not in a failed state.
     Since control-plane-only argument is not specified, this will upgrade the control plane AND all nodepools to version 1.22. Continue? (y/N): y

  $ az aks show --resource-group prodpublick8s --name prodpublick8s --output table
  Name             Location    ResourceGroup              KubernetesVersion    CurrentKubernetesVersion    ProvisioningState    Fqdn
  ---------------  ----------  -------------------------  -------------------  --------------------------  -------------------  --------------------------------------------------
  temp-privatek8s  eastus2     prodpublick8s  1.22                 1.22.6                      Succeeded            temp-privatek8s-dns-c5f4426a.hcp.eastus2.azmk8s.io
```
         
         
  We did hit a problem with PVC and PV as the **new** parameter "secretNamespace" was not provided by our helm definition the secrets were not in the correct Namespace.
         Here is the PR that added it: https://github.com/jenkins-infra/helm-charts/pull/158
         
         
  - [x] Close maintenance windows in status (<https://github.com/jenkins-infra/status/pull/...>)
  - [x] Write post mortem for what went wrong (if needed)
- [x] Expected downtimes: none

### Issues Met during This Upgrade
        


#### Persistent Volumes of kind "Azurefile"

ERROR : we had to specify the namespace where the SAS token lives, otherwises Kubernetes searches on the default namespace.
        
as per the AKS CSI for Kube 1.22+ (ref. jenkins-infra/helpdesk#2930), we have to specify the namespace where the SAS token lives, otherwises Kubernetes searches on the default namespace.

Same hotfix as hotfix(ldap) specify secretNamespace for ldap-backup PV #158 but for the release.ci.jenkins.io environment
Long term fix: as Upgrade to Kubernetes 1.22 helpdesk#2930 will underline, we need to check all of the PVs to use the latest CSI driver, or at least patch all the remaining azurefile PVs.
For the record, we had to set the replica count to 0 on the pods in order to delete the PVC AND PV to apply the SecretNameSpace

### Template Email Mailing List: 

```
Hello everybody,

We’ll do an AKS clusters upgrade from 1.XX to 1.YY this FULL DATE, starting at XX:00 UTC.
See more at https://github.com/jenkins-infra/helpdesk/issues/....
No outage is expected, but there could be an impact (1-2 min outage) on the following services if there is an unexpected upgrade issue:
  accounts.jenkins.io
  beta.accounts.jenkins.io
  customize.jenkins.io
  get.jenkins.io
  incrementals.jenkins.io
  infra.ci.jenkins.io
  javadoc.jenkins.io
  jenkins-wiki-exporter.jenkins.io…

The Infra Team
```

