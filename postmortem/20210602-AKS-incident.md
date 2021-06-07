---
tags: post-mortem
state: open
project: infrastructure
---
<!-- markdownlint-disable MD013 -->

# 2021-06-02 - AKS Incident (Service Principal Expiration)

**! This document is open to feedback until the 14th of June 2021 then this document will be archived on [jenkins-infra/documentation](https://github.com/jenkins-infra/documentation) !**

## Chronology

* 2021-06-02 - 17:53 UTC: The [Azure Service Principal Credential](https://docs.microsoft.com/en-us/azure/active-directory/develop/app-objects-and-service-principals) expired.
* 2021-06-02 - 18:02 UTC: Errors about pods unable to mount volumes, or scaled up nodes unable to obtain network virtual interface from CNI start to appear
  * It includes the LTS process cf. [LTS 2.289.1 post-mortem report](https://hackmd.io/@jenkins-infra/H1OPUzI9_)
* 2021-06-03 - 15:15 UTC: The Service principal credential is renewed by the infra team, leading to an automatic an unstoppable operation on the AKS cluster to restart all nodes
* 2021-06-03 - 16:35 UTC: All operation have finished successfully, all Kubernetes-hosted applications had been restart and no more errors found in logs or metrics

## Impacts

* Unable to schedule a pod with a Persistent Volume from 2021-06-02 - 18:02 UTC until 2021-06-03 - 15:30 UTC: failing builds when a pod with PVC is used on trusted.ci.jenkins.io, infra.ci.jenkins.io and release.ci.jenkins.io in this window time
* ldap.jenkins.io was unable due to a full restart from 2021-06-03 - 15:17 UTC until 2021-06-03 - 15:19 UTC
* plugins.jenkins.io answered HTTP/500 on 33% of the requests from 2021-06-03 - 15:15 UTC until 2021-06-03 - 16:35 UTC

## Technical Elements

* Status updates:
  * Open: <https://github.com/jenkins-infra/status/pull/27>
  * Close: <https://github.com/jenkins-infra/status/pull/28>
* [AKS Documentation to renew the SP](https://docs.microsoft.com/en-us/azure/aks/update-credentials)
* @dduportal does not have the full authorization set on Azure to reset the SP: to ensure a prompt back to normal, @olblak did the reset operation.
  * At least @MarkEWaite and a 3rd person seems to have the rights: no bus factor here

## What Went Wrong?

* The Service Principal Credential is used by AKS nodes to authenticate to the Azure API to obtain rights to mount a file system volume (either Azure Blob, CIFS, etc.), private network accesses or Azure LoadBalancer creation/deletion
* The AKS UI Console helped to get the error while investigating the root cause of [LTS 2.289.1 post-mortem report](https://hackmd.io/@jenkins-infra/H1OPUzI9_)
* Reseting the SP requires a full node restart, which is done with a standard Kubernetes cordon -> drain -> update -> reboot process

## Proposal Improvements

### Short term

* SP reset (done during the incident)
* Add a PagerDutty automatic check to trigger an alert a week before, like what we have for certificates
  * <https://github.com/Cj-Scott/Get-AppRegistrationExpiration> from @zvW_c6ROSOOuJDTOracA7Q 
      * @olblak  : @dduportal , It would be nice to create a JIRA ticket to keep track of this work because I really doubt that we will work on this anytime soon

### Medium-term

* Find a way to have a team-level calendar to add such events as a manual operation to reset a credential
  * Google calendar? A bot?
      * @olblak: It's not the first that a team calendar would be useful for infra work.

### Long-term

* @zvW_c6ROSOOuJDTOracA7Q mentioned that "managed identities" can be used instead of SP, which has the benefit of auto-renewing.
    * Same here, it would be good to  create a Jira ticket to keep track of this potential work.
