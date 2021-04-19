[![hackmd-github-sync-badge](https://hackmd.io/EDpvZx9ZS2GWgHHqWFRDfQ/badge)](https://hackmd.io/EDpvZx9ZS2GWgHHqWFRDfQ)

# Jenkins Kubernetes Upgrade

**2021-04-14**

This document contains information about how to upgrade Kubernetes cluster maintained by the Jenkins infrastructure project.

## Template
This is the default list of requirements that we should consider before every Cluster upgrade. Feel free to add any suggestions you may have.

### Requirements

- Create [Jira](https://issues.jenkins.io) Ticket 
- Announce on:
  - [Jenkins status page](https://github.com/jenkins-infra/status)
  - IRC, channel #jenkins-infra on freenode
  - [Jenkins Infrastructure and Dev Mailing list](https://groups.google.com/g/jenkins-infra)
  - [Twitter](https://twitter.com/jenkinsci/), in case of major upgrade
- Review Release Changelogs 
  - [Kubernetes](https://github.com/kubernetes/kubernetes/tree/master/CHANGELOG)
  - [AKS](https://github.com/Azure/AKS/blob/master/CHANGELOG.md)
- Review and merge important pull request from [jenkins-infra/charts](https://github.com/jenkins-infra/charts)
- Block k8smgmnt job on infra.ci
- Pagerduty
  - Notify the on call person
- Dependency update
  - Bump kubectl docker image [docker-helmfile](https://github.com/jenkins-infra/docker-helmfile) to the targeted kubernetes controller version
  - Apply in the jenkins-infra/charts repo: https://github.com/jenkins-infra/charts/pull/1056
  
- Search for deprecated kubernetes resources
  - [Pluto](https://github.com/FairwindsOps/pluto) running `pluto detect-helm -owide`

---

## Upgrade

###  1.17.x  to 1.18.x

Releated to publick8s

- Jira ticket: [INFRA-2944](https://issues.jenkins.io/browse/INFRA-2944)
- Review Release Changelogs 
  - [Kubernetes](https://github.com/kubernetes/kubernetes/tree/master/CHANGELOG) @dduportal 
  - [AKS](https://github.com/Azure/AKS/blob/master/CHANGELOG.md) @olblak 

- k8smngmt (charts and block)
  - ⚠️ [PR#1000](https://github.com/jenkins-infra/charts/pull/1000) to be tested 
 
- Bump `kubectl` version 
  - [PR#11](https://github.com/jenkins-infra/docker-helmfile/pull/11) 
 
#### Risks 

No major risk indentified

#### Notes

For futur releases, we must keep in mind the following elements

- [nginx-ingress](https://github.com/kubernetes/ingress-nginx/tree/master/charts/ingress-nginx#migrating-from-stablenginx-ingress)
- New ingress syntax introduced in 1.18
- Resources that need upgrading (fetched using pluto)

| Name | Namespace | Kind | Version | Replacement | Deprecated | Deprecated In | Removed | Removed In |
| ---- | --------- | ---- | ------- | ----------- | ---------- | ------------- | ------- | ---------- |
| cert-manager/certificaterequests.cert-manager.io | cert-manager | CustomResourceDefinition | apiextensions.k8s.io/v1beta1 | apiextensions.k8s.io/v1 | true | v1.16.0 | false | v1.22.0 |
| cert-manager/certificates.cert-manager.io | cert-manager | CustomResourceDefinition | apiextensions.k8s.io/v1beta1 | apiextensions.k8s.io/v1 | true | v1.16.0 | false | v1.22.0 |
| cert-manager/challenges.acme.cert-manager.io | cert-manager | CustomResourceDefinition | apiextensions.k8s.io/v1beta1 | apiextensions.k8s.io/v1 | true |  v1.16.0 | false | v1.22.0 |
| cert-manager/clusterissuers.cert-manager.io | cert-manager | CustomResourceDefinition | apiextensions.k8s.io/v1beta1 | apiextensions.k8s.io/v1 | true | v1.16.0 | false | v1.22.0 |
| cert-manager/issuers.cert-manager.io | cert-manager | CustomResourceDefinition | apiextensions.k8s.io/v1beta1 | apiextensions.k8s.io/v1 | true | v1.16.0 | false | v1.22.0 |
| cert-manager/orders.acme.cert-manager.io | cert-manager | CustomResourceDefinition | apiextensions.k8s.io/v1beta1 | apiextensions.k8s.io/v1 | true | v1.16.0 | false | v1.22.0 |
| cert-manager/cert-manager-webhook | cert-manager | MutatingWebhookConfiguration | admissionregistration.k8s.io/v1beta1 | admissionregistration.k8s.io/v1 | true | v1.16.0 | false | v1.22.0 |
| default-release-nexus/default-release-nexus | release | Ingress | extensions/v1beta1 | networking.k8s.io/v1 | true | v1.14.0 | false | v1.22.0 |
