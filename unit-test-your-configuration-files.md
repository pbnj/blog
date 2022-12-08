---
date: 2020-02-20
tags: ["kubernetes", "docker", "configuration", "unit testing", "conftest"]
---

# Unit Testing Configuration Files

![pile of paper garbage](https://images.unsplash.com/photo-1516992654410-9309d4587e94?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=2700&q=80)
_Infrastructure as Code. Photo courtesy:
[@Bass Emmen](https://unsplash.com/@mediavormgever)_

## Table of Contents

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Overview](#overview)
- [Getting Started](#getting-started)
  - [Dockerfile](#dockerfile)
  - [Kubernetes](#kubernetes)
- [Next Steps](#next-steps)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Overview

The era of Infrastructure-as-Code (IaC) has unlocked tremendous developer
productivity and agility features. Now, as an Engineer, we can declare our
infrastructure and environments as structured data in configuration files, such
as Terraform templates, Dockerfiles, and Kubernetes manifests.

However, this agility and speed of provisioning and configuring infrastructure
comes with a high risk of bugs in the form of misconfigurations.

Fortunately, we can solve this problem just as we can solve for other bugs in
our products, by writing **unit tests**.

One such tool that can help us unit test our configuration files is
[`conftest`](https://github.com/instrumenta/conftest). What is unique about
`conftest` is that it uses
[Open-Policy-Agent](https://github.com/open-policy-agent/opa) (OPA) and a policy
language, called
[Rego](https://www.openpolicyagent.org/docs/latest/policy-language/) to
accomplish this.

This might appear difficult at first, but it will start to make sense.

Let's explore 2 use-cases where we can test our configurations!

## Getting Started

First, some prerequisites:

- [`conftest`](https://github.com/instrumenta/conftest):
  - macOS: `brew install instrumenta/instrumenta/conftest`
- **(Optional)** [`opa`](https://github.com/open-policy-agent/opa):
  - macOS: `brew install opa`

### Dockerfile

Let's say we want to prevent some images and/or tags (e.g. `latest`).

We need to create a simple Dockerfile:

```Dockerfile
FROM kalilinux/kali-linux-docker:latest

ENTRYPOINT ["echo"]
```

Now, we need to create our first unit test file, let's call it `test.rego`, and
place it in a directory, let's call it `policy` (this is configurable).

```rego
package main

disallowed_tags := ["latest"]
disallowed_images := ["kalilinux/kali-linux-docker"]

deny[msg] {
        input[i].Cmd == "from"
        val := input[i].Value
        tag := split(val[i], ":")[1]
        contains(tag, disallowed_tags[_])

        msg = sprintf("[%s] tag is not allowed", [tag])
}

deny[msg] {
        input[i].Cmd == "from"
        val := input[i].Value
        image := split(val[i], ":")[0]
        contains(image, disallowed_images[_])

        msg = sprintf("[%s] image is not allowed", [image])
}
```

Assuming we are in the right directory, we can test our Dockerfile:

```sh
$ ls
Dockerfile      policy/

$ conftest test -i Dockerfile ./Dockerfile
FAIL - ./Dockerfile - [latest] tag is not allowed
FAIL - ./Dockerfile - [kalilinux/kali-linux-docker] image is not allowed
```

Just to be sure, let's change this Dockerfile to pass the test:

```Dockerfile
# FROM kalilinux/kali-linux-docker:latest
FROM debian:buster

ENTRYPOINT ["echo"]
```

```sh
$ ls
Dockerfile      policy/

$ conftest test -i Dockerfile ./Dockerfile
PASS - ./Dockerfile - data.main.deny
```

"It works! But I don't understand how," I hear you thinking to yourself.

Let's break the Rego syntax down:

- `package main` is a way for us to put some rules that belong together in a
  namespace. In this case, we named it `main` because `conftest` defaults to it,
  but we can easily do something like `package docker` and then run
  `conftest test -i Dockerfile --namespace docker ./Dockerfile`
- `disallowed_tags` & `disallowed_images` are just simple variables that hold an
  array of strings
- `deny[msg] { ... }` is the start of the deny rule and it means that the
  Dockerfile should be rejected and the user should be given an error message
  `msg` if the conditions in the body (i.e. `{ ... }`) are true
- Expressions in the body of the deny rule are treated as logical AND. For
  example:

  ```rego
  1 == 1                    # IF 1 is equal to 1
  contains("foobar", "foo") # AND "foobar" contains "foo"
                            # This would trigger the deny rule
  ```

- `input[i].Cmd == "from"` checks if the Docker command is `FROM`. `input[i]`
  means we can have multiple Dockerfiles being tested at once. This will iterate
  over them
- The next 2 lines are assignments just to split a string and store some data in
  variables
- `contains(tag, disallowed_tags[_])` will return true if the `tag` we obtained
  from the Dockerfile contains one of the `disallowed_tags`. `array[_]` syntax
  means iterate over values
- `msg := sprinf(...)` creates the message we want to tell our user if this deny
  rule is triggered
- The second `deny[msg]` rule checks that the image itself is not on the
  blocklist.

### Kubernetes

Let's say we want to ensure that all pods are running as a non-root user.

We need to create our deployment

```sh
$ mkdir -p kubernetes
$ cat <<EOF >./kubernetes/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.7.9
          ports:
            - containerPort: 80
EOF
```

Now, we need to create our unit test:

```sh
$ mkdir -p ./kubernetejjjs/policy
$ cat <<EOF >./kubernetes/policy/test.rego
package main

name := input.metadata.name

deny[msg] {
  input.kind == "Deployment"
  not input.spec.template.spec.securityContext.runAsNonRoot

  msg = sprintf("Containers must run as non root in Deployment %s. See: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/", [name])
}
EOF
```

And, let's run it:

```sh
conftest test -i yaml ./kubernetes/deployment.yaml
FAIL - ./kubernetes/deployment.yaml - Containers must run as non root in Deployment nginx-deployment. See: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
```

This is a bit more straightforward:

- Get the `metadata.name` from the `input` (which is the Kubernetes Deployment
  yaml file)
- Create a deny rule that is triggered if:
  - `input.kind` is `Deployment` and
  - `securityContext.runAsNonRoot` is not set
- Return an error message to the user that containers must run as non-root and
  point them to the docs.

## Next Steps

So, where to go from here?

The Rego language is vast and it can take a bit to wrap your head around how it
works. You can even send and receive HTTP requests inside Rego.

I recommend reading the docs to learn more about Rego's capabilities:

- [Reference sheet](https://www.openpolicyagent.org/docs/latest/policy-reference/)
- [Cheat sheet](https://www.openpolicyagent.org/docs/latest/policy-cheatsheet/)

I also barely scratched the surface with `conftest` in this blog post. The
repository has a nice list of
[examples](https://github.com/instrumenta/conftest#examples) that you should
peruse at your leisure. `conftest` even supports sharing policies via uploading
OPA bundles to OCI-compliant registries, e.g. `conftest push ...`,
`conftest pull ...`.

Lastly, if you have any questions, the OPA community is friendly and welcoming.
Feel free to join the `#conftest` channel in
[OPA Slack](https://slack.openpolicyagent.org/).

Happy coding!
