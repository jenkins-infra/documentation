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

- [x] Proceed to the upgrade
  
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

    - [x] Starting the upgrade in Azure (prodpublick8s)

        ```console
        # == prodpublick8s upgrade ==
        # Go to https://release.ci.jenkins.io/prepareShutdown/

        # TODO: Deactivate "Kubernetes Management" job on infra.ci.jenkins.io


        # available upgrades
        $ az aks get-upgrades --resource-group prodpublick8s --name prodpublick8s --output table
          Name     ResourceGroup    MasterVersion    Upgrades
          -------  ---------------  ---------------  ---------------
          default  prodpublick8s    1.22.15          1.23.8, 1.23.12
        # upgrade 
        $ az aks upgrade \
            --resource-group prodpublick8s \
            --name prodpublick8s \
            --kubernetes-version 1.23.12

            Kubernetes may be unavailable during cluster upgrades.
            Are you sure you want to perform this operation? (y/N): y
            Since control-plane-only argument is not specified, this will upgrade the control plane AND all nodepools to version 1.23.12. Continue? (y/N): y

        $ az aks show --resource-group prodpublick8s --name prodpublick8s --output table
            Name           Location    ResourceGroup    KubernetesVersion    CurrentKubernetesVersion    ProvisioningState    Fqdn
            -------------  ----------  ---------------  -------------------  --------------------------  -------------------  ------------------------------------------------
            prodpublick8s  eastus2     prodpublick8s    1.23.12              1.23.12                     Succeeded            prodpublick8s-dns-bdcd7389.hcp.eastus2.azmk8s.io
        ```

    - [ ] Close maintenance windows in status (<https://github.com/jenkins-infra/status/pull/189>)
    - [x] Write post mortem for what went wrong (if needed)

- [x] Expected downtimes: none

### Issues Met during This Upgrade
  None
  
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
  javadoc.jenkins.io
  jenkins-wiki-exporter.jenkins.io…

  infra.ci.jenkins.io

The Infra Team
```
