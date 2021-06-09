---
tags: post-mortem
state: open
project: infrastructure
---

# 2021-06-03 - repo.jenkins-ci.org authentication issue

**! This document is open to feedback until the 15th of June 2021 then this document will be archived on [jenkins-infra/documentation](https://github.com/jenkins-infra/documentation) !**

## Chronology

* 2021-06-07 12:37:00 - Incidents reported on status.jenkins.io
* 2021-06-07 16:37:00 - Two IPs have been manually added to ldap.jenkins.io configuration.
* 2021-06-08 07:37:00 - This [PR](https://github.com/jenkins-infra/charts/pull/1238) persist in time the hotfix we applied.

## Impacts

Nobody could authenticate with repo.jenkins-ci.org.
This prevented maintainers to release new artifacts version on repo.jenkins-ci.org

## Technical Elements

Due to a recent upgrade done by JFrog, the outbound IP that communicate with our ldap service, changed. This had for consequence that ldap queries were blocked on the Jenkins side. 

## What went wrong?

## Proposal Improvements

### Short Term

We added the two new IP to our configuration [here](https://github.com/jenkins-infra/charts/pull/1238)

### Medium Term

We should establish a communication channel between the Jenkins project and Jfrog other than Twitter

### Long Term