---
tags: maintenance
---
<!-- markdownlint-disable MD013 MD036-->

# Jenkins AKS Kubernetes Upgrade to 1.23

<https://docs.microsoft.com/en-us/azure/aks/upgrade-cluster?tabs=azure-cli>

## 1.22.x  to 1.23.x

Related to `prodpublicks8` and `temp-privatek8s` clusters

- [x] Issue: <https://github.com/jenkins-infra/helpdesk/issues/3053>

- [x] Pre-Flight checks: 
  - [x] Changelog Scrapping:
    - [x] [Urgent Upgrade Notes for 1.23](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.23.md#urgent-upgrade-notes), nothing to report
    - [x] [Azure Changelog](https://github.com/Azure/AKS/blob/master/CHANGELOG.md): nothing to report
      - [x] [Selected quotes](https://github.com/jenkins-infra/helpdesk/issues/3053#issuecomment-1292119216)

- [x] Announces:
  - [x] `status.jenkins.io`: <https://github.com/jenkins-infra/status/pull/210>
  - [x] IRC
  - [x] ML (jenkins-infra@googlegroups.com, jenkinsci-dev@googlegroups.com) [template bellow]

- [ ] Proceed to the upgrade
  
  - Upgrade Procedure:
    - [x] Ensure we have access to all clusters in kube-config
    - [x] Pagerduty: Notify the on call person
    - [x] Stopping the k8s management job
    - [x] Starting the upgrade in Azure (temp-prodk8s)

        ```console
        # == temp-privatek8s upgrade ==
        # add a message to prepare the shutdown by using: https://infra.ci.jenkins.io/prepareShutdown/

        # login
        $ az login

        # available upgrades
        $ az aks get-upgrades --resource-group prod-jenkins-private-prod --name temp-privatek8s --output table
          Name     ResourceGroup              MasterVersion    Upgrades
          -------  -------------------------  ---------------  ---------------
          default  prod-jenkins-private-prod  1.22.15          1.23.8, 1.23.12
        # upgrade full, control-plane AND nodes
        $ az aks upgrade \
            --resource-group prod-jenkins-private-prod \
            --name temp-privatek8s \
            --kubernetes-version 1.23.12

            Kubernetes may be unavailable during cluster upgrades.
            Are you sure you want to perform this operation? (y/N): y
            Since control-plane-only argument is not specified, this will upgrade the control plane AND all nodepools to version 1.23.12. Continue? (y/N): y
            ....
        # Before upgrade
        $ kubectl get nodes                         
          NAME                                  STATUS   ROLES   AGE   VERSION
          aks-infracipool-10382320-vmss00004g   Ready    agent   15m   v1.22.15
          aks-linuxpool-29019427-vmss00000m     Ready    agent   24h   v1.22.15
          aks-linuxpool-29019427-vmss00001g     Ready    agent   20m   v1.22.15
          aks-systempool-13530583-vmss000002    Ready    agent   45h   v1.22.15
        
        # After upgrade
        $ kubectl get nodes      
          NAME                                  STATUS   ROLES   AGE     VERSION
          aks-infracipool-10382320-vmss00004g   Ready    agent   15m     v1.23.12
          aks-infracipool-10382320-vmss00004h   Ready    agent   2m39s   v1.23.12
          aks-linuxpool-29019427-vmss00000m     Ready    agent   13m     v1.23.12
          aks-linuxpool-29019427-vmss00001g     Ready    agent   10m     v1.23.12
          aks-systempool-13530583-vmss000002    Ready    agent   14m     v1.23.12
        ```

    - [ ] Starting the upgrade in Azure (prodpublick8s)

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
        Here is the PR that added it: <https://github.com/jenkins-infra/helm-charts/pull/158>

    - [ ] Close maintenance windows in status (<https://github.com/jenkins-infra/status/pull/189>)
    - [ ] Write post mortem for what went wrong (if needed)

- [ ] Expected downtimes: none

### Issues Met during This Upgrade

#### Persistent Volumes of kind "Azurefile"

ERROR : we had to specify the namespace where the SAS token lives, otherwises Kubernetes searches on the default namespace.

as per the AKS CSI for Kube 1.22+ (ref. <https://github.com/jenkins-infra/helpdesk/issues/2930>), we have to specify the namespace where the SAS token lives, otherwises Kubernetes searches on the default namespace.

Same hotfix as hotfix(ldap) specify secretNamespace for ldap-backup PV <https://github.com/jenkins-infra/kubernetes-management/pull/2551> but for the release.ci.jenkins.io environment
Long term fix: as Upgrade to Kubernetes 1.22 <https://github.com/jenkins-infra/helpdesk/issues/2930> will underline, we need to check all of the PVs to use the latest CSI driver, or at least patch all the remaining azurefile PVs.
For the record, we had to set the replica count to 0 on the pods in order to delete the PVC AND PV to apply the SecretNameSpace

### Template Email Mailing List

```text
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
