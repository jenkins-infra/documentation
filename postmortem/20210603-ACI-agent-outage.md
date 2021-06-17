---
tags: post-mortem
state: open
project: infrastructure
---
<!-- markdownlint-disable MD013 -->

# 2021-06-03 - ACI Agent Outage for ci.jenkins.io

**! This document is open to feedback until the 14th of June 2021 then this document will be archived on [jenkins-infra/documentation](https://github.com/jenkins-infra/documentation) !**

## Chronology

* 2021-06-03 - 15:35 UTC: ACI agents allocations start failing with containers started, but the agent process fails to connect to the controller with "Handshake failed" connection errors
* 2021-06-03 - 15:41 UTC: ACI Agent allocation is not possible because of the limit of 800 deployments reached
* 2021-06-03 - 19:30 UTC: ci.jenkins.io is put on shutdown mode to limit the amount of incoming requests
* 2021-06-04 - 05:30 UTC: start of investigations (EU morning) leading to multiple pipeline kills and build queue forced cleanup
* 2021-06-04 - 07:30 UTC: all plugins of ci.jenkins.io are upgraded, leading to instance restart (3 times)
* 2021-06-04 - ~08:30 UTC: ci.jenkins.io starts to cleanup the ACI agents by itself after a plugin update
* 2021-06-04 - 09:02 UTC: ci.jenkins.io is able to allocate ACI agents normally and builds are resuming
* 2021-06-08 - 5AM UTC. ACI agent provisioning failures continue and impact the Jenkins core pull requests. Builds fail with "Error: missing workspace /home/jenkins/workspace/Core_jenkins_PR-5563 on aci-maven-11-j6tsw"

## Impacts

* No builds involving ACI agents on ci.jenkins.io from 2021-06-03 - 15:41 UTC until 2021-06-04 09:02 - UTC
* No builds **at all** on ci.jenkins.io from 2021-06-03 - 19:30 UTC until 2021-06-04 09:02 - UTC
* Build queue cleaned up with deletion of all build from 2021-06-03 - 15:41 UTC until 2021-06-04 09:02 - UTC
* Acceptance-test build random errors related to EC2 agent disconnection during the whole day of 2021-06-04 as a consequence of the EC2 plugin upgrade failing to cleanup properly dangling instances

## Technical Elements

* All plugins where up-to-date the 2021-06-04 at 09:00 UTC
* Status links:
  * Open: <https://github.com/jenkins-infra/status/pull/28>
  * Close: <https://github.com/jenkins-infra/status/pull/29>

## What Went Wrong?

* We don't really know: the timeslot is close to the AKS SP issue, but there should be no relation between both pieces of infrastructure (ci.jenkins.io runs on a VM, not on Kubernetes). There could be a link around Azure API that was throttled to explain the non-deletion of ACI agent deployment, but it does not explain why these agents started to fail
* ci.jenkins.io logs were reporting the following error message during the incident:

    ```text
    2021-06-04 08:36:26.094+0000 [id=54]    WARNING hudson.slaves.NodeProvisioner#lambda$update$6: Unexpected exception encountered while provisioning agent aci-maven-77g69
    java.lang.NullPointerException
            at com.microsoft.jenkins.containeragents.aci.AciCloud.getAzureClient(AciCloud.java:83)
            at com.microsoft.jenkins.containeragents.aci.AciService.createDeployment(AciService.java:52)
            at com.microsoft.jenkins.containeragents.aci.AciContainerTemplate.provisionAgents(AciContainerTemplate.java:128)
            at com.microsoft.jenkins.containeragents.aci.AciCloud.lambda$provision$1(AciCloud.java:109)
            at jenkins.util.ContextResettingExecutorService$2.call(ContextResettingExecutorService.java:46)
            at jenkins.security.ImpersonatingExecutorService$2.call(ImpersonatingExecutorService.java:80)
            at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
            at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128)
            at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628)
            at java.base/java.lang.Thread.run(Thread.java:829)
    ```

  * Thanks to the help of @zvW_c6ROSOOuJDTOracA7Q , this error was tracked down to a code that was calling the Azure credential for the azure-client used to dial with the remote ACI API
  * Both credentials and agent XML files on the JENKINS_HOME were referencing a plugin version greated than the actual one: consequence of the [previous ACI outage ending up in downgrading plugins](https://hackmd.io/XBFjAgkFQ1mjiKPnsW2EXw)
  * While trying to save (on the UI as there is no CasC configuration) the Azure credential or the Cloud agent configuration, HTTP/500 errors were answered, with 2 kind of messages: related to EC2-configuration, or missing class for Azure-commons: thas is the reason why we decided to upgrade all plugins to be sure there was no more issue related to bad dependency tree
* After upgrading all plugins, a successful restart and a successful "save" for the cloud agents configurations, then everything went back to normal

## Proposal Improvements

### Short term

* Remove the plugin [Azure Commons](https://plugins.jenkins.io/azure-commons/) as proposed by @zvW_c6ROSOOuJDTOracA7Q [INFRA-3010](https://issues.jenkins.io/browse/INFRA-3010)

### Medium-term

* Mixes the agent container workload to not depends only on ACI (Kubernetes agent could be used) [INFRA-2918](https://issues.jenkins.io/browse/INFRA-2918)

### Long-term

* Add a pager duty alert when 80% of the 800 deployment ACI threshold is reached - [INFRA-3011](https://issues.jenkins.io/browse/INFRA-3011)
* Add a sanity check that compares config file versions and installed plugin versions?  Alternately, don't upgrade plugins without a configuration backup and don't downgrade plugins without comparing current configuration files with backup configuration files? [Sounds like a mandatory point for ci.jenkins.io as code](https://issues.jenkins.io/browse/INFRA-2917)
