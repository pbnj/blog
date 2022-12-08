---
date: 2018-07-19
tags: ["docker", "kubernetes", "container", "security"]
---

# Docker Security Best Practices

![container](https://upload.wikimedia.org/wikipedia/en/4/41/Stefan_Beese%27s_Eco_Shipping_Container_Lounge_by_Melissa_Carrier.jpg)

## Table of Contents

- [Docker Security Best Practices](#docker-security-best-practices)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [The Host](#the-host)
  - [Docker Hardening Standard](#docker-hardening-standard)
  - [Docker Engine](#docker-engine)
  - [Container Privileges](#container-privileges)
  - [Static Analysis](#static-analysis)
  - [Runtime Security](#runtime-security)
  - [Conclusion](#conclusion)

## Overview

Containers have revolutionized the tech industry in recent years for many
reasons. Because of their properties of encapsulating dependencies into a
portable container image, many organizations have adopted them as the primary
method of developing, building, and deploying production applications.

As a Software Engineer and a Security Engineer, I cannot even begin to express
my excitement for the game-changing potential of containers and orchestrators,
namely Kubernetes. However, due to the chaotic (good) nature of open-source and
the speed by which projects move and evolve, many organizations are simply
unable or unequipped to properly secure these technologies.

This article aims to provide a list of common security mistakes and security
best-practices/recommendations in 2018.

In the next article, I will offer the same insights for Kubernetes.

**Legend:**

| Icon | Meaning         |
| ---: | :-------------- |
|   ❌ | Not Recommended |
|   🗒️ | Rationale       |
|   ✅ | Recommendation  |

---

## The Host

❌ Running Docker on an unsecured, unhardened host

🗒️ Docker is only as secure as the underlying host

✅ Make sure you follow
[OS security best-practices](https://downloads.cisecurity.org/) to harden your
infrastructure. If you dole out `root` access to every user in your
organization, then it doesn't matter how secure Docker is.

---

## Docker Hardening Standard

✅ The Center for Internet Security (CIS) puts out documents detailing security
best-practices, recommendations, and actionable steps to achieve a hardened
baseline. The best part:
[they're free](https://downloads.cisecurity.org/download-issues/benchmarks).

✅ Better yet,
[docker-bench-security](https://github.com/docker/docker-bench-security) is an
automated checker based on the CIS benchmarks.

```sh
# recommended
$ docker run \
    -it \
    --net host \
    --pid host \
    --userns host \
    --cap-add audit_control \
    -e DOCKER_CONTENT_TRUST=$DOCKER_CONTENT_TRUST \
    -v /var/lib:/var/lib \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v /usr/lib/systemd:/usr/lib/systemd \
    -v /etc:/etc --label docker_bench_security \
    docker/docker-bench-security
```

---

## Docker Engine

Docker Engine is an API that listens for incoming requests and, in turn,
interfaces with the underlying host kernel to accomplish its job. Docker Engine
supports communications on 3 different sockets: `unix`, `tcp`, and `fd`.

❌ Running Docker Engine (aka the Docker daemon, aka `dockerd`) on `tcp` or any
networked socket

🗒️ If anyone can reach the networked socket that Docker is listening on, they
potentially have access to Docker and, since Docker needs to run as root, to the
underlying host

✅ The default docker behavior today is the safest assumption, which is to
listen on a `unix` socket

```sh
# not recommended
$ dockerd -H "tcp://1.2.3.4:8080"

# recommended
$ dockerd -H "unix:///var/run/docker.sock"
```

---

❌ Mounting the Docker socket into the container

🗒️ Mounting `/var/run/docker.sock` inside the container is a common, yet very
dangerous practice. An attacker can execute any command that the docker service
can run, which generally provides access to the whole host system as the docker
service runs as root.

✅ Short of just saying "Don't mount the docker socket", carefully consider the
use-cases that require this giant loophole. For example, many tutorials for
running a Jenkins master in a container will instruct you to mount the docker
socket so that Jenkins can spin up other containers to run your tests in. This
is dangerous as that means anyone can execute any shell commands from Jenkins to
gain unauthorized access to sensitive information or secrets (e.g. API tokens,
environment variables) from other containers, or launch privileged containers
and mount `/etc/shadow` to extract all users' passwords.

```sh
# not recommended
$ docker run -it -v /var/run/docker.sock:/var/run/docker.sock ubuntu /bin/bash
```

---

## Container Privileges

❌ Running privileged containers

🗒️ Containers would have full access to the underlying host

✅ If needed by a container, grant it only the specific capabilities that it
needs

```sh
# not recommended
$ docker run -d --privileged ubuntu

# recommended
$ docker run -d --cap-add SYS_PTRACE ubuntu
```

---

❌ Running containers as root users

🗒️ This is a system administration standard best-practice. There is little to no
reason for running software in containers as `root`.

✅ Run containers as non-root users

```sh
# not recommended (runtime example)
$ docker run -d ubuntu sleep infinity
$ ps aux | grep sleep
root ... sleep infinity

# recommended (runtime example)
$ docker run -d -u 1000 ubuntu sleep infinity
$ ps aux | grep sleep
1000 ... sleep infinity

# recommended (build-time example)
FROM ubuntu:latest
USER 1000
```

---

## Static Analysis

❌ Pulling and running containers from public registries

🗒️ Recently, security researchers found 17 cryptomining containers on Docker Hub

✅ Scan container images to detect and prevent containers with known
vulnerabilities or malicious packages from getting deployed on your
infrastructure

- Open source tools in this space are [anchore](https://anchore.com/) and
  [clair](https://github.com/coreos/clair)

---

✅ Sign container images

- [Docker Content Trust](https://docs.docker.com/engine/security/trust/content_trust/)
  guarantees the integrity of the publisher and the integrity of the contents of
  a container image, thus establishing trust

---

## Runtime Security

✅ Attach `seccomp`, `apparmor`, or `selinux` profiles to your containers

- Security profiles, like
  [`seccomp`](https://docs.docker.com/engine/security/seccomp/),
  [`apparmor`](https://docs.docker.com/engine/security/apparmor/), and `selinux`
  add stronger security boundaries around the container to prevent it for making
  a SYSCALL it is not explicitly allowed to make

---

✅ Monitor, detect, and alert on anomalous, suspicious, and malicious container
behavior

- Open source tools in this space include
  [Sysdig Falco](https://github.com/draios/falco)

---

✅ Consider running containers in a container runtime sandbox, like
[gvisor](https://github.com/google/gvisor)

- Container runtime sandbox add an even stronger security boundary around your
  containers at runtime

---

## Conclusion

Container technology is not inherently more or less secure than traditional
virtualization technologies. Containers are enabled by Linux features, such as
namespace isolation and cgroups to control to control system resources. Securing
container workloads and the systems underneath them require an understanding of
Linux as a platform.
