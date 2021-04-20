---
tags: runbook, pkg, pkg.jenkins.io, pkg.origin.jenkins.io
project: infrastructure
---

# Runbook: Distribution's Packaging Service

[![hackmd-github-sync-badge](https://hackmd.io/UjmXampzQtSJT_OLHbJaMA/badge)](https://hackmd.io/UjmXampzQtSJT_OLHbJaMA)

## Description

pkg.jenkins.io -> package repositories indexes serves through a Fastly CDN

pkg.origin.jenkins.io -> direct DNS to the AWS VM, which also serves update-center, build machine for Deb/CentOS packages, and mirrorbrain (ancested of mirrorbit), and 4 PgSQL databases for mirrorbrain

get.jenkins.io -> mirroring service based on mirrorbits, which goal is to redirect to a mirror, or to the fallback


fallback.get.jenkins.io -> mirrorbit-file -> fallback, direct apache serve


## Source Code

## Links

## Connection

## How To

Data Redis: Azure managed
