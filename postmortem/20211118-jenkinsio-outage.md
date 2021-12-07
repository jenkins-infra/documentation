---
tags: post-mortem
state: open
project: infrastructure
---
<!-- markdownlint-disable MD013 -->

# 2021-11-18 - jenkins.io outage

**! This document is open to feedback until the 30th of november 2021, then it will be archived on [jenkins-infra/documentation/postmortem/20211118-jenkinsio-outage.md](https://github.com/jenkins-infra/documentation/blob/main/postmortem/20211118-jenkinsio-outage.md) !**

## Chronology

* 2021-11-18 - 10:42 UTC: The nginx helm chart from `4.0.6` to `4.0.8` (patch version) is successfully deployed. 
  * It includes creating a new webhook admission rule that forbids any ingress annotation containing an Nginx configuration block (e.g. any `{}`).
  * Side effect: the ingress rule for jenkins.io (also www.jenkins.io, origin.jenkins.io and www.origin.jenkins.io) is deleted by Kubernetes because it's not compliant.
* 2021-11-18 - 11:07 UTC: Fastly start to decache some pages of jenkins.io because it cannot update it from our Kubernetes cluster (ingress rule deleted means that the pages answers HTTP/404 to fastly)
* 2021-11-18 - 11:32 UTC: a user reports in the Gitter channel jenkinsci/jenkins that https://www.jenkins.io/doc/pipeline/steps/ is answering HTTP/503 errors (message "Guru meditation" from Fastly's varnish)
* 2021-11-18 - 11:49 UTC: status.jenkins.io is reporting the incident
* 2021-11-18 - 13:40 UTC: a manual fix is applied to ensure that jenkins.io is back, while disabling all of our automated system. It consists on rollbacking the nginx helm chart to its previous version `4.0.6` + manually deleting the webhook adminission systems. 
* 2021-11-18 - 15:30 UTC: after applying a first batch of changes to the jenkinsio configuration in Kubernetes, the Nginx helm chart is re-deployed to `4.0.8` leading in a 2nd jenkins.io outage
* 2021-11-18 - 17:11 UTC: all of the required fixes have been applied: the automated system redeploys jenkins.io's helm chart: the website is back and fastly is ordered to decache all pages. End of outage


## Impacts

* jenkins.io was unavailable (answering HTTP/503 errors) during the 2 following time windows:
  * 2021-11-18 - 11:07 UTC => 13:40 UTC (02h 37min of outage)
  * 2021-11-18 - 15:30 UTC => 17:11 UTC (01h 41min of outage)

## Technical Elements

* Change for Nginx 4.0.8 upgrade: https://github.com/jenkins-infra/charts/pull/1708
* Changes with fixes (failed and successfuls) for the outage:
  * jenkins-infra/charts:
    * (remove forbidden annotations) https://github.com/jenkins-infra/charts/pull/1724
    * (rollback nginx ingress to `4.0.6`) https://github.com/jenkins-infra/charts/commit/aa9f5293ae38b6958a4fde6a9196333d1cb68908
    * (upgrade jenkinsio helm chart to a last version known to work as expected with nginx `4.0.8`) https://github.com/jenkins-infra/charts/commit/03d55efab4a69fc2c947b56495073363775426cc
    * (Nginx ingress back to `4.0.8`) https://github.com/jenkins-infra/charts/commit/93a6228823b404ee5b3108ec72db5ab52db4738a

## What Went Wrong?

* We trusted a "patch" version of an Helm chart on a sensitive component (the Ingress controller) without checking the changelog deep dive
* We did not have a monitoring on origin.jenkins.io that reported the errors once visible

## Proposal Improvements

### Short term

* Always check the changelog deep dive when updating the ingress controllers + pair to have at least 2 different eyes in real time
  * Already done in https://github.com/jenkins-infra/charts/pull/1736/files to upgrade the nginx-ingress to `4.0.9` successfully

### Medium-term

* Improve monitoring to ensure that an outage on www.origin.jenkins.io detects such an issue
  * Q: Is there a monitoring for this domain?
      * A: We don't: https://github.com/jenkins-infra/datadog/search?q=origin.jenkins.io only mentions "pkg.origin". Great point: we could make it faster to detect outages on this by adding a custom datadog monitoring. 
  * Ensure that this monitoring must present an header "Host" set to "www.jenkins.io" to NOT be redirected to jenkins.io (e.g. to Fastly)
  * Add a set of pages to be checked, not only the front page (such as pipeline steps page)

### Long-term

* Q: Would a prototype of pkg.origin.jenkins.io have allowed us to detect this issue before a production deployment?
    * A: (Assuming you meant "a prototype of www.origin.jenkins.io") if we had a complete staging cluster with the same DNS setup, then yes. Maybe we could have a "k3s" ephemeral end to end testing before deploying to production though. That's a nice improvement to think about for the next quarter!
