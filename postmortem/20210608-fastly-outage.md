---
tags: post-mortem
state: open
project: infrastructure
---
<!-- markdownlint-disable MD013 -->

# 2021-06-08 - Fastly Outage

**! This document is open to feedback until the 17th of June 2021 then this document will be archived on [jenkins-infra/documentation](https://github.com/jenkins-infra/documentation) !**

## Chronology

* 2021-06-08 - 09:47 UTC - As described in the [Fasty June 8 Outage Summary](https://www.fastly.com/blog/summary-of-june-8-outage), a global outage on the CDN service provided by Fastly happens worldwide.
* 2021-06-08 - 10:11 UTC - The Jenkins team start to be aware about the outage (IRC message, Pager Dutty alerts, user feedbacks on different channels)
* 2021-06-08 - 10:30 UTC - After 2 failures ti update status.jenkins.io, impacted by the outage, an escalation starts to have a cross-channel communication. Oleg helps by synchronizing and evaluating communications
* 2021-06-08 - 11:30 UTC - Synchronized communication is done through [Twitter](https://twitter.com/jenkinsci/status/1402226409364393984?s=20) and the [mailing list](https://groups.google.com/g/jenkinsci-dev/c/8stJD5tbwGs)
* 2021-06-08 - 11:35 UTC - Fastly mitigates the issue
* 2021-06-08 - 11:38 UTC - All the impacted Jenkins services are back to normal


## Impacts

* [www.jenkins.io](https://www.jenkins.io) was completely unavailable from 2021-06-08 - 09:47 UTC until 2021-06-08 - 11:38 UTC
* [pkg.jenkins.io](https://pkg.jenkins.io) was down so nobody could install or upgrade Jenkins core using Debian/Redhat/Suse package manager
* [plugins.jenkins.io](https://plugins.jenkins.io) was down so nobody could access plugin information

## Technical Elements

* Status updates (even if it failed):
  * Open <https://github.com/jenkins-infra/status/pull/33>
  * Close <https://github.com/jenkins-infra/status/pull/34>

## What Went Wrong?

~~* We could not  update the status page because of the Fastly outage 
status.jenkins.io ourtage: Both GH Pages and Netlify depend on Fastly: should we move to a static custom HTML site to avoid dependency to Fastly (e.g. is it really a problem if this happens once per year? Twice? More? Compared to the frequency of occurences of other incidents)
Infra team has no way to make announcements~~

* We couldn't update the status page because of a [typo](https://github.com/jenkins-infra/status/pull/33/files#diff-3d09ddbed98b4c76eaa84d1db85c9cc2f7391e6fa8b7efd68d7271a9c3a188cbR9) in an incident description

## Proposal Improvements

### Short term

* The only thing we could have done to mitigate the incident was to update the DNS record to by pass Fastly and redirect traffics to the service origin such pkg.origin.jenkins.io for pkg.jenkins.io. Considering that we don't have SLA, imho it's better to just wait that Fastly solves its incident`
* Improve the status.jenkins.io update:
  * Add a CI provess before Netlify (for instance ensure hugo builds without error) to avoid Pebkac as the on I did (@dduportal) and not rely of self manual test before commit
 
### Medium-term

* Write down the incident response process to improve the collaboration and limit intruptions for the people fixing issues during outages
  * Includes creating a process to allow powsting on the Jenkins Twitter account for big outages (Tweetdeck sharing? other?)
* Improve the status.jenkins.io update:
  * Should be use PR that are slowing down the process?
  * Auto-generate issues incident to avoid errors in filing the correct fields
  * Maybe using a bot to simplify the update?

### Long-term

* Create runbook for outage on status.jenkins.io: it is a static site, it should be easy to deploy it efficiently on an azure webapp (proposal from @timja), a static VM, whatever and only "switch" the DNS
