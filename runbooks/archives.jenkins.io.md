---
tags: runbook, archives, archives.jenkins.io
project: infrastructure
---

# Runbook: Jenkins archive mirror

## Description

`archives.jenkins.io` is a service storing every artifact created since the creation of the Jenkins project including old Hudson versions.
This service is available on [HTTP/HTTPS](https://archives.jenkins.io).

This service is currently hosted on the Rackspace account and managed using Puppet as defined on [jenkins-infra/jenkins-infra](https://github.com/jenkins-infra/jenkins-infra/blob/staging/dist/role/manifests/okra.pp)

Data published on archives.jenkins.io are generated from various scripts located on pkg.origin.jenkins.io.

Once [INFRA-2961](https://issues.jenkins.io/browse/INFRA-2961) is implemented, we will be able to use `archives.jenkins.io` as a source location for third mirrors and an additional mirror from [get.jenkins.io](https://get.jenkins.io/windows/2.251/jenkins.msi.sha256?mirrorlist)

## Issues

Issues related to `archives.jenkins.io` are reported on [Jira](https://issues.jenkins.io).
Please use the component "[archives](https://issues.jenkins.io/issues/?jql=project%20%3D%20INFRA%20AND%20component%20%3D%20archives)"

## Links

* [Jira - issues](https://issues.jenkins.io/issues/?jql=project%20%3D%20INFRA%20AND%20component%20%3D%20archives)
* [Web](https://archives.jenkins.io)

## Connection

This service is only reachable using SSH.
You need a key defined in the main puppet [hieradata file](https://github.com/jenkins-infra/jenkins-infra/blob/staging/hieradata/common.yaml)
under the key `accounts::<jenkins_username>`.
More information on the [README](https://github.com/jenkins-infra/jenkins-infra/README.md) about how to contribute
