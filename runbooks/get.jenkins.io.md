---
tags: runbook, get.jenkins.io, mirrorbits
project: infrastructure
---

# Runbook: Geographical Download Redirector (get.jenkins.io)

## Description

get.jenkins.io is a "Geographical Download Redirector" based on [mirrorbits](https://github.com/etix/mirrorbits).

Its goal is to redirect an HTTP file download request to a mirror which is closed geographically, or to answer the request using [Fallback File Download Service (fallback.get.jenkins.io)](./fallback.get.jenkins.io.md).

This service is hosted in the Kubernetes cluster [`publick8s` (Azure Kubernetes Service)](https://github.com/jenkins-infra/azure/blob/master/plans/publick8s.tf), and managed using a GitOps process using Helmfile (see source code section below).

⚠️ Any Kubernetes resource (deployment, pod, etc.) named `mirrorbit-files` is NOT associated to `get.jenkin.io`, but to the [Fallback File Download Service (fallback.get.jenkins.io)](./fallback.get.jenkins.io.md).

## Source Code

* Helmfile manifest: https://github.com/jenkins-infra/charts/blob/master/helmfile.d/mirrorbits.yaml
* Helm Chart: https://github.com/jenkins-infra/charts/tree/master/charts/mirrorbits
* Installation (Helm) Values: https://github.com/jenkins-infra/charts/blob/master/config/default/mirrorbits.yaml
    * Ingress rule for `get.jenkins.io`: https://github.com/jenkins-infra/charts/blob/master/config/default/mirrorbits.yaml#L28


## Links


## Connection

* Check that you have a valid `kubeconfig` to access [`publick8s` (Azure Kubernetes Service)](https://github.com/jenkins-infra/azure/blob/master/plans/publick8s.tf)
* Use the namespace `mirrorbits`

## How To
