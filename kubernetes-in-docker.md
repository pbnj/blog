---
date: 2019-05-12
tags: ["kubernetes", "docker", "container", "development"]
draft: true
---

# Kubernetes in Docker

## Table of Contents

- [Kubernetes in Docker](#kubernetes-in-docker)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [A Kind Introduction](#a-kind-introduction)
  - [Use Cases](#use-cases)

## Overview

When it comes to developing applications and services for Kubernetes, developers
have no shortage of tools that range across a specturm. Tools like
[Azure Draft](https://github.com/Azure/draft) and
[GoogleCloud Skaffold](https://github.com/GoogleContainerTools/skaffold) require
developers to push code to remote environments, while tools like minikube
require virutal machines.

But, in this article, I would like to focus on a developer-friendly and
versatile tool: **kind**

## A Kind Introduction

`kind`, short for Kubernetes-in-Docker, is a tool that brings up a Kubernetes
cluster in Docker containers. Docker has revolutionized software development by
essentially packaging up applications into distributable images that we can run
any where. `kind` brings these developer ergonomics to Kubernetes and container
orchestration, thus unlocking really interesting dynamics that I would like to
explore next.

## Use Cases

So, what can we do with kind?

I am glad you asked.

[Duffie Cooley](https://mauilion.dev) recently wrote about a
[great use-case for kind](https://mauilion.dev/posts/kind-k8s-testing/) where he
tests a Kubernetes Pull-Request. It is definitely worth a read as he dives into
so much detail.

In addition to being a tool for Kubernetes contributors, application developers
can use `kind` to package up a number of applications together and deploy them
as a single unit.
