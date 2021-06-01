---
tags: post-mortem
state: closed
project: infrastructure
---
<!-- markdownlint-disable MD013 -->

# 2021-05-29 - Azure Container Instance outage on ci.jenkins.io

## Timeline - May 29, 2021

* 16:37 Denver - Mark Waite updated plugins on ci.jenkins.io, restarted ci.jenkins.io
* 16:55 Denver - Updated JDK 8 tool definition from 8u242 to 8u292
* 17:15 Denver - Confirmed builds with JDK 8u292 tools working on amd64, arm64, s390x, and popc64le
* 17:39 Denver - Started Jenkins core build, detected that maven and maven-11 agents were not being allocated
* 17:41 Denver - System log for ACI agents reports `Cannot provision: template for label maven is not available now, because it failed to provision last time.`
* 17:50 Denver - Downgraded Azure container plugin to previously installed version, restarted Jenkins
* 18:00 Denver - Downgraded plugin reports `NoSuchMethodError`, likely indicator that it requires an API in one of the 50+ plugins that were upgraded at 16:37 Denver time
* 18:01 Denver - Updated Azure container plugin to latest release
* 18:17 Denver - Confirmed problem is still visible after upgrade
* 18:19 Denver - Record plugin file system times in case rollback needed
* 19:14 Denver - Identify candidate plugins for rollback
  * azure-sdk from 12.vc102aedd3c66 down to 4.vcb202d9010c1
  * azure-credentials from 182.v3ccd4a755864 down to 177.v816b81058012
  * azure-vm-agents from 780.v50d067d02f76 down to 774.v0cee503baa25
  * windows-azure-storage from 358.v5c001416d74f down to 355.v4da08e72a251
  * azure-container-agents from 207.v3ad9931bf69e down to 201.v2afdce22b4cf
* Review dependencies of those plugins
  * Used plugin installation manager to install a local copy of those 5 plugins at their previous versions as confirmation that they should be workable at their previous versions as a group.  Not a perfect check, but worth the check.
* 20:01 Denver - Downgrade those 5 plugins and restart ci.jenkins.io
* 20:03 Denver - Restart is complete, Manage Plugins shows those 5 plugins have upgrades available
* 20:09 Denver - Outage resolved, ACI agents are allocated on ci.jenkins.io

## Plugins that were upgraded at 16:37 Denver time

```
-rw-r--r-- 1 jenkins jenkins  4200464 May 29 22:37 plugins/jacoco.jpi
-rw-r--r-- 1 jenkins jenkins   380040 May 29 22:37 plugins/kubernetes.jpi
-rw-r--r-- 1 jenkins jenkins  3091463 May 29 22:37 plugins/ldap.jpi
-rw-r--r-- 1 jenkins jenkins    86675 May 29 22:37 plugins/blueocean-commons.jpi
-rw-r--r-- 1 jenkins jenkins   277630 May 29 22:37 plugins/blueocean-jwt.jpi
-rw-r--r-- 1 jenkins jenkins    39703 May 29 22:37 plugins/command-launcher.jpi
-rw-r--r-- 1 jenkins jenkins   468323 May 29 22:37 plugins/cloudbees-bitbucket-branch-source.jpi
-rw-r--r-- 1 jenkins jenkins  8089625 May 29 22:37 plugins/subversion.jpi
-rw-r--r-- 1 jenkins jenkins   102390 May 29 22:37 plugins/blueocean-rest.jpi
-rw-r--r-- 1 jenkins jenkins    14124 May 29 22:37 plugins/blueocean-jira.jpi
-rw-r--r-- 1 jenkins jenkins    86109 May 29 22:37 plugins/workflow-multibranch.jpi
-rw-r--r-- 1 jenkins jenkins   308017 May 29 22:37 plugins/branch-api.jpi
-rw-r--r-- 1 jenkins jenkins    35870 May 29 22:37 plugins/blueocean-pipeline-scm-api.jpi
-rw-r--r-- 1 jenkins jenkins  2363613 May 29 22:37 plugins/jenkins-design-language.jpi
-rw-r--r-- 1 jenkins jenkins  1076110 May 29 22:37 plugins/blueocean-core-js.jpi
-rw-r--r-- 1 jenkins jenkins  1413507 May 29 22:37 plugins/blueocean-web.jpi
-rw-r--r-- 1 jenkins jenkins   967614 May 29 22:37 plugins/blueocean-rest-impl.jpi
-rw-r--r-- 1 jenkins jenkins   192974 May 29 22:37 plugins/blueocean-pipeline-api-impl.jpi
-rw-r--r-- 1 jenkins jenkins    66662 May 29 22:37 plugins/blueocean-github-pipeline.jpi
-rw-r--r-- 1 jenkins jenkins  2745123 May 29 22:37 plugins/blueocean-dashboard.jpi
-rw-r--r-- 1 jenkins jenkins   703681 May 29 22:37 plugins/blueocean-personalization.jpi
-rw-r--r-- 1 jenkins jenkins 33137044 May 29 22:37 plugins/azure-sdk.jpi
-rw-r--r-- 1 jenkins jenkins    58121 May 29 22:37 plugins/azure-credentials.jpi
-rw-r--r-- 1 jenkins jenkins  5947031 May 29 22:37 plugins/jira.jpi
-rw-r--r-- 1 jenkins jenkins    28517 May 29 22:37 plugins/display-url-api.jpi
-rw-r--r-- 1 jenkins jenkins  2068729 May 29 22:37 plugins/pipeline-utility-steps.jpi
-rw-r--r-- 1 jenkins jenkins   776815 May 29 22:37 plugins/plugin-util-api.jpi
-rw-r--r-- 1 jenkins jenkins   486809 May 29 22:37 plugins/font-awesome-api.jpi
-rw-r--r-- 1 jenkins jenkins   261324 May 29 22:37 plugins/azure-vm-agents.jpi
-rw-r--r-- 1 jenkins jenkins   957359 May 29 22:37 plugins/credentials.jpi
-rw-r--r-- 1 jenkins jenkins 16099656 May 29 22:38 plugins/analysis-model-api.jpi
-rw-r--r-- 1 jenkins jenkins   226188 May 29 22:38 plugins/mercurial.jpi
-rw-r--r-- 1 jenkins jenkins    54575 May 29 22:38 plugins/blueocean-git-pipeline.jpi
-rw-r--r-- 1 jenkins jenkins   127789 May 29 22:38 plugins/matrix-auth.jpi
-rw-r--r-- 1 jenkins jenkins    34438 May 29 22:38 plugins/pubsub-light.jpi
-rw-r--r-- 1 jenkins jenkins 11342134 May 29 22:38 plugins/warnings-ng.jpi
-rw-r--r-- 1 jenkins jenkins   472289 May 29 22:38 plugins/windows-slaves.jpi
-rw-r--r-- 1 jenkins jenkins  1360650 May 29 22:38 plugins/cvs.jpi
-rw-r--r-- 1 jenkins jenkins  2425761 May 29 22:38 plugins/ec2.jpi
-rw-r--r-- 1 jenkins jenkins   634219 May 29 22:38 plugins/workflow-cps.jpi
-rw-r--r-- 1 jenkins jenkins   839972 May 29 22:38 plugins/timestamper.jpi
-rw-r--r-- 1 jenkins jenkins  1102491 May 29 22:38 plugins/windows-azure-storage.jpi
-rw-r--r-- 1 jenkins jenkins    61490 May 29 22:38 plugins/blueocean-config.jpi
-rw-r--r-- 1 jenkins jenkins    92076 May 29 22:38 plugins/blueocean-bitbucket-pipeline.jpi
-rw-r--r-- 1 jenkins jenkins    23581 May 29 22:38 plugins/blueocean-events.jpi
-rw-r--r-- 1 jenkins jenkins  1057838 May 29 22:38 plugins/blueocean-pipeline-editor.jpi
-rw-r--r-- 1 jenkins jenkins    15198 May 29 22:38 plugins/blueocean-i18n.jpi
-rw-r--r-- 1 jenkins jenkins     8105 May 29 22:38 plugins/blueocean.jpi
-rw-r--r-- 1 jenkins jenkins   308385 May 29 22:38 plugins/github-branch-source.jpi
-rw-r--r-- 1 jenkins jenkins    44276 May 29 22:38 plugins/structs.jpi
-rw-r--r-- 1 jenkins jenkins    96874 May 29 22:38 plugins/workflow-durable-task-step.jpi
-rw-r--r-- 1 jenkins jenkins    35277 May 29 22:38 plugins/kubernetes-credentials.jpi
-rw-r--r-- 1 jenkins jenkins    66978 May 29 22:38 plugins/ansicolor.jpi
-rw-r--r-- 1 jenkins jenkins    99985 May 29 22:38 plugins/groovy.jpi
-rw-r--r-- 1 jenkins jenkins  2337135 May 29 22:38 plugins/configuration-as-code.jpi
-rw-r--r-- 1 jenkins jenkins   189508 May 29 22:38 plugins/script-security.jpi
-rw-r--r-- 1 jenkins jenkins   822801 May 29 22:38 plugins/support-core.jpi
-rw-r--r-- 1 jenkins jenkins    98403 May 30 00:00 plugins/azure-container-agents.jpi

```
