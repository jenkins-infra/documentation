---
tags: maintenance
---

# Nginx-Ingress upgrade

[INFRA-2743](https://issues.jenkins.io/browse/INFRA-2743)

The Jenkins Infrastructure project uses two Nginx-Ingress controllers in front of every websites hosted on the Kubernetes cluster.
One for public services and a second one internal services only reachable from the Jenkins VPN.


Helm V2 charts are [deprecated](https://github.com/helm/charts/#%EF%B8%8F-deprecation-and-archive-notice) in favor of the helm v3.
They are now hosted in a different location.
The Nginx Ingress controller is the remaining chart used by the Jenkins infrastructure project which has to be upgraded.

[migrating from stable/nginx-ingess](https://github.com/kubernetes/ingress-nginx/tree/master/charts/ingress-nginx#migrating-from-stablenginx-ingress)

## Risks

The major risks during this upgrade is to stop redirecting HTTP/HTTPS traffics to the following services:

* Public services:
  * [www.jenkins.io](https://www.jenkins.io)
  * [plugins.jenkins.io](https://plugins.jenkins.io)
  * [javadoc.jenkins.io](https://javadoc.jenkins.io)
  * [get.jenkins.io](https://get.jenkins.io)
  * [fallback.get.jenkins.io](https://fallback.get.jenkins.io)
  * [reports.jenkins.io](https//reports.jenkins.io)
  * [uplink.jenkins.io](https://uplink.jenkins.io)
  * [customize.jenkins.io](https://customize.jenkins.io)
  * [incrementals.jenkins.io](https://incrementals.jenkins.io)
  * [beta.accounts.jenkins.io](https://beta.accounts.jenkins.io)
  * [mirror.azure.jenkins.io](https://mirror.azure.jenkins.io)
  * [jenkins-wiki-exporter.jenkins.io](https://jenkins-wiki-exporter.jenkins.io)
  * [archives.azure.jenkins.io](https://archives.azure.jenkins.io)

* Private services:
  * [release.ci.jenkins.io](https://release.ci.jenkins.io)
  * [infra.ci.jenkins.io](https://infra.ci.jenkins.io)
  * [admin.accounts.jenkins.io](https://admin.accounts.jenkins.io)
  * [grafana.publick8s.jenkins.io](https://grafana.publick8s.jenkins.io/)

## Requirements

"Requirements" is the list of actions which must be done prior the maintenance window.

* [x] Announce maintenance
  * [x] status.jenkins.io
  * [x] Infra meeting
  * [x] Mailing list
* [x] Reduce DNS TTL to 10min
  * [x] public.aks.jenkins.io
  * [x] private.aks.jenkins.io
  * [x] ~~public.aks.jenkins-ci.org~~
  * [x] ~~private.aks.jenkins-ci.org~~

## ToDo

"ToDo" is the list of actions which must be done during the maintenance window.

* [x] Notify on-call people
* [x] Migrate Private Nginx controller
  * [x] Stop k8s management job on infra.ci
  * [x] Deploy a new temporary private Nginx controller
  * [x] 20 min before next operation, we must reduce DNS TTL to 1min.
    * [x] private.aks.jenkins.io
  * [x] Update DNS traffic to use the new ingress controller IP
    * [x] private.aks.jenkins.io
    * [x] ~~private.aks.jenkins-ci.org~~

  * [x] Wait until no traffic arrive on the old ingress controller
  * [x] Delete old ingress controller `helm delete -n kube-system private-nginx-ingress`
  * [x] Deploy the new ingress controller
  * [x] Redirect back dns traffic from the temporay controller to the official one
  * [x] Increase DNS TTL back to 60min
    * [x] private.aks.jenkins.io
* [x] Migrate Public Nginx controller
  * [x] Stop k8s management job on infra.ci
  * [x] Deploy a new temporary private Nginx controller
  * [x] 20 min before next operation, we must reduce DNS TTL to 1min.
    * [x] public.aks.jenkins.io
    * [x] jenkins.io
  * [x] Update DNS traffic to use the new ingress controller IP
    * [x] public.aks.jenkins.io
    * [x] jenkins.io (APEX)
    * [x] ~~public.aks.jenkins-ci.org~~
  * [x] Wait until no traffic arrive on the old ingress controller
  * [x] Delete old ingress controller `helm delete -n kube-system public-nginx-ingress`
  * [x] Deploy the new ingress controller
  * [x] Redirect back dns traffic from the temporay controller to the official one
  * [x] Increase DNS TTL back to 60min
    * [x] public.aks.jenkins.io
    * [x] jenkins.io (APEX)
* [x] Start k8s management job on infra.ci
* [x] Remove tmp ingress controller
  * [x] PR on jenkins-infra/charts
  * [x] Manual deletion of the controllers once the PR is merged, with `helm delete -n private-nginx-ingress tmp-private-nginx-ingress --dry-run` and `helm delete -n public-nginx-ingress tmp-public-nginx-ingress --dry-run` (test with the dry run, adn then apply)
* [x] Close maintenance window on status.jenkins.io
* [x] Celebrate

## Notes

* Automate [404](https://github.com/jenkins-infra/404) docker image workflow - [INFRA-2967](https://issues.jenkins.io/browse/INFRA-2967)

* Nginx (new) features to check later
    * Enable [brotli](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/#enable-brotli) => check the impact on subservices (gzip in brotli? meh)
    * Enable [Datadog](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/#datadog-collector-host) 
    * Enable Prometheus metrics
* Increase the number of controller from 1 to 2, done
* Discussion about provisioning a new public IP for this operation, and keep it after. Decided to NOT do that, to avoid breaking usages of companies whom have allowed the IP on their firewall: if we change the public IP it will break these usages.

* Rename ingress rule from 'kubernetes.io/ingress.class' to 'ingressClassName' ([doc](https://kubernetes.io/docs/concepts/services-networking/ingress/#deprecated-annotation) and [blog](https://kubernetes.io/blog/2020/04/02/improvements-to-the-ingress-api-in-kubernetes-1.18/))
  * `kubectl get ingress -A -o yaml   | grep 'kubernetes.io/ingress.class'`
  * [jenkins-infra/charts](https://github.com/jenkins-infra/charts/search?l=YAML&q=kubernetes.io%2Fingress.class)

* ! New ingress controller implement ingressClassName instead of 'kubernetes.io/ingress.class' [INFRA-2969](https://issues.jenkins.io/browse/INFRA-2969)
* Set private ingress as default controller using the right annotation 'isDefaultClass' (ref. https://kubernetes.io/docs/concepts/services-networking/ingress/#default-ingress-class) - [INFRA-2970](https://issues.jenkins.io/browse/INFRA-2970)
* Improve ingress deployment by setting node selector to enforce a deployment on a linux node as underlined by https://docs.microsoft.com/en-us/azure/aks/ingress-basic#create-an-ingress-controller - [INFRA-2971](https://issues.jenkins.io/browse/INFRA-2971)
* No DNS change needed for domain jenkins-ci.org (there are only CNAME to `*.jenkins.io` records)
* DNS change is required for private and public `*.jenkins.io` records, but also for the apex domain `jenkins.io` which must be a `IN` record pointing to the public ingress