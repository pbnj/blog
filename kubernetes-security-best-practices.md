---
date: 2018-07-19
tags: ["docker", "kubernetes", "container", "security"]
---

# Kubernetes Security Best Practices

![cargo ship](https://www.logisticsmgmt.com/images/LM1710_SUP_Ocean_wideImage2.jpg)

## Table of Contents

- [Kubernetes Security Best Practices](#kubernetes-security-best-practices)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Secure Baseline](#secure-baseline)
  - [Authentication](#authentication)
  - [Authorization](#authorization)
  - [Admission Controls](#admission-controls)
  - [Impersonation](#impersonation)
  - [Pod Security Policies](#pod-security-policies)
    - [Full Pod Security Policy Example](#full-pod-security-policy-example)
  - [Network Policies](#network-policies)
  - [Additional Security Measures](#additional-security-measures)
  - [Additional References](#additional-references)
  - [Conclusion](#conclusion)

## Overview

So, you are all-in on container technology. You may be spinning up a container
for your application and another container for the database locally on your
machine, but then what? You need systems to build your containers. You need
systems to deploy your containers. You need systems to run your containers. How
would you manage containers at this scale?

Enter, Kubernetes!

Kubernetes is the new Application Server. Kubernetes provides an API interface
for dev teams, ops teams, and even security teams to interact with applications
and the platform.

Kubernetes is a fast-moving open-source project with constant progress being
made. So, my goal in this article is to cover some common security mistakes I
have observed and offer some general best-practices around securing Kubernetes
clusters and workloads.

**Legend:**

| Icon | Meaning         |
| ---- | --------------- |
| ❌   | Not Recommended |
| 🗒️   | Rationale       |
| ✅   | Recommendation  |
| ⚠️   | Warning         |

## Secure Baseline

✅ Ensure that your underlying hosts are hardened and secure. I recommend CIS
benchmarks as a starting point.

✅ Ensure that Docker itself is configured per security best-practices. Check
out my previous article:
[Docker Security Best-Practices](https://dev.to/petermbenjamin/docker-security-best-practices-45ih)

✅ Ensure that you're starting off with Kubernetes with a secure baseline.

- Center for Internet Security (CIS) maintains
  [documentation](https://downloads.cisecurity.org/) available for free.
- The fine folks at Aqua Security also open-sourced an automated checker based
  on CIS recommendations. Check it out:
  [kube-bench](https://github.com/aquasecurity/kube-bench)

```sh
# recommended - on master node
$ kubectl run                         \
    --rm                              \
    -it                               \
    kube-bench-master                 \
    --image=aquasec/kube-bench:latest \
    --restart=Never                   \
    --overrides="{ \"apiVersion\": \"v1\", \"spec\": { \"hostPID\": true, \"nodeSelector\": { \"kubernetes.io/role\": \"master\" }, \"tolerations\": [ { \"key\": \"node-role.kubernetes.io/master\", \"operator\": \"Exists\", \"effect\": \"NoSchedule\" } ] } }"               \
    -- master                         \
    --version 1.8

# recommended - on worker nodes
$ kubectl run                         \
    --rm                              \
    -it                               \
    kube-bench-node                   \
    --image=aquasec/kube-bench:latest \
    --restart=Never                   \
    --overrides="{ \"apiVersion\": \"v1\", \"spec\": { \"hostPID\": true } }" \
    -- node                           \
    --version 1.8
```

---

## Authentication

Most interactions with Kubernetes are done by talking to the control plane,
specifically the **kube-apiserver** component of the control plane. Requests
pass through 3 steps in the kube-apiserver before the request is served or
rejected: Authentication, Authorization, and Admission Control. Once requests
pass these 3 steps, kube-apiserver communicates with
[Kubelets](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/)
over the network. Therefore, Kubelets also have to check authentications and
authorizations as well.

The behavior of `kube-apiserver` and `kubelet` can be controlled or modified by
launching them with certain command-line flags. The full list of supported
command-line flags are documented here:

- [kube-apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)
- [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/)

Let's examine some common Kubernetes authentication security mistakes:

❌ By default, anonymous authentication is enabled

🗒️ Kubernetes allows anonymous authentication out-of-the-box. There are a
combination of `kube-apiserver` settings that render anonymous authentication
safe, like [`--authorization-mode=RBAC`](#authorization) because you would need
to explicitly grant RBAC privileges to `system:anonymous` user and
`system:unauthenticated` group. **Note**: Granting RBAC privileges to `*` user
or `*` group do not include anonymous users.

✅ Disable anonymous authentication by passing the `--anonymous-auth=false` flag

```sh
# not recommended - on master node
$ kube-apiserver       \
    <... other flags>  \
    --anonymous-auth=true

# recommended - on master node
$ kube-apiserver       \
    <... other flags>  \
    --anonymous-auth=false

# not recommended - on worker nodes
$ kubelet             \
    <... other flags> \
    --anonymous-auth=true

# recommended - on worker nodes
$ kubelet             \
    <... other flags> \
    --anonymous-auth=false
```

---

❌ Running `kube-apiserver` with `--insecure-port=<PORT>`

🗒️ In older versions of Kubernetes, you could run `kube-apiserver` with an API
port that does not have any protections around it

✅ Disable insecure port by passing the `--insecure-port=0` flag. In recent
versions, this has been disabled by default with the intention of completely
deprecating it

```sh
# not recommended
$ kube-apiserver         \
    <... other flags>    \
    --insecure-port=6443

# recommended
$ kube-apiserver      \
    <... other flags> \
    --insecure-port=0
```

---

✅ Prefer **OpenID Connect** or **X509 Client Certificate**-based authentication
strategies over the others when authenticating users

🗒️ Kubernetes supports different authentication strategies:

- **X509 client certs**: decent authentication strategy, but you'd have to
  address renewing and redistributing client certs on a regular basis
- **Static Tokens**: avoid them due to their non-ephemeral nature
- **Bootstrap Tokens**: same as static tokens above
- **Basic Authentication**: avoid them due to credentials being transmitted over
  the network in cleartext
- **Service Account Tokens**: should not be used for end-users trying to
  interact with Kubernetes clusters, but they are the preferred authentication
  strategy for applications & workloads running on Kubernetes
- **OpenID Connect (OIDC) Tokens**: best authentication strategy for end users
  as OIDC integrates with your identity provider (e.g. AD, AWS IAM, GCP IAM
  ...etc)

```sh
# recommended - OIDC
$ kube-apiserver                                                        \
    <... other flags>                                                   \
    --oidc-issuer-url="https://domain/.well-known/openid-configuration" \
    --oidc-client-id="example"

# recommended - X509 cert
$ kube-apiserver                               \
    <... other flags>                          \
    --client-ca-file=/path/to/ca.crt           \
    --tls-cert-file=/path/to/server.crt        \
    --tls-private-key-file=/path/to/server.key
```

---

## Authorization

❌ By default, authorization mode is to always authorize all requests to
`kube-apiserver`

✅ Enable
[Role-Based Access Controls](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

🗒️ Kubernetes supports different authorization strategies, but **Role-Based
Access Control** (RBAC) was introduced in v1.8, which allows administrators to
define what users and service accounts can or cannot do across Kubernetes
clusters as a whole or within specific Kubernetes
[namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

```sh
# not recommended
$ kube-apiserver      \
    <... other flags> \
    --authorization-mode=AlwaysAllow

# recommended
$ kube-apiserver      \
    <... other flags> \
    --authorization-mode=RBAC
```

---

❌ By default, the `default` service account is automatically mounted into the
file system of all containers in Kubernetes

🗒️ The `default` service account is a valid service account token that can be
used to query the Kubernetes API as an authenticated workload. With RBAC
enabled, this is still a problem, but not as big as a problem as without RBAC
enabled. Without RBAC, this token can be used to do virtually anything in the
kubernetes cluster

✅ Disable auto-mounting of the `default` service account token

```sh
# recommended
$ kubectl patch serviceaccount default -p "automountServiceAccountToken: false"
```

---

## Admission Controls

Admission controllers are pieces of code that intercept requests to the
Kubernetes API in the 3rd and last step of the authentication/authorization
process. The full list of admission control options are documented here:

- [Admission Controllers](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)

---

✅ Configure admission control to deny privilege escalation via launching
interactive shells on or attaching to privileged containers

🗒️ There are some valid use-cases where you need to run privileged containers to
interact with the kernel of the underlying host, but this means users can
potentially `kubectl attach` or `kubectl exec` into those privileged containers.

```sh
# recommended
kube-apiserver       \
    <...other flags> \
    --admission-control=...,DenyEscalatingExec
```

---

✅ Configure admission control to enable Pod Security Policies

🗒️ Pod Security Policies are security rules that pods have to abide by in order
to be accepted and scheduled on your cluster.

⚠️ **Make sure you have PodSecurityPolicy objects (i.e. yaml files) ready to be
applied once you turn this on, otherwise no pod will be scheduled.** See
[Pod Security Policy](#pod-security-policy) recommendation in this article for
examples.

```sh
# recommended
$ kube-apiserver      \
    <... other flags> \
    --addmission-control=...,PodSecurityPolicy

# example PodSecurityPolicy
$ kubectl create -f- <<EOF
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: example
spec:
  privileged: false  # Don't allow privileged pods!
EOF

# let's try to request a privileged pod to be scheduled on our cluster
$ kubectl create -f- <<EOF
apiVersion: v1
kind: Pod
metadata:
  name:      pause
spec:
  containers:
    - name:  pause
      image: k8s.gcr.io/pause
EOF

# throws error as expected
Error from server (Forbidden): error when creating "STDIN": pods "privileged" is forbidden: unable to validate against any pod security policy: [spec.containers[0].securityContext.privileged: Invalid value: true: Privileged containers are not allowed]
```

---

✅ Configure admission control to always pull images

🗒️ Kubernetes will cache container images that have been pulled from any
registry (private or public). Any cached container images can be re-used by any
pod on the cluster. Therefore, pods could gain unauthorized access to
potentially sensitive information. Forcing Kubernetes to always pull images will
require that the credentials needed to pull down images from private registries
are provided by each requesting resource

```sh
# recommended
$ kube-apiserver      \
    <... other flags> \
    --admission-control=...,AlwaysPullImages
```

---

## Impersonation

Kubernetes has a feature that allows any user to impersonate any other user on
the Kubernetes cluster. This feature is nice for debugging, but can have
unintended security implications if not controlled properly.

To demonstrate the implications:

```sh
$ kubectl drain test-node
Error from server (Forbidden): User "foo" cannot get nodes at the cluster scope. (get nodes test-node)

$ kubectl drain test-node --as=admin --as-group=system:admins
node "test-node" cordoned
node "test-node" drained
```

You can read more about it here:

- [User Impersonation](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#user-impersonation)

✅ Limit who can impersonate and what they can do as impersonated users

```sh
# not recommended
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: impersonator
rules:
# allows users to impersonate any "users", "groups", and "serviceaccounts"
- apiGroups: [""]
  resources: ["users", "groups", "serviceaccounts"]
  verbs: ["impersonate"]


# recommended
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: limited-impersonator
rules:
# Can impersonate the group "developers"
- apiGroups: [""]
  resources: ["groups"]
  verbs: ["impersonate"]
  resourceNames: ["developers"]
```

---

## Pod Security Policies

Pod Security Policies define a set of conditions that a pod must abide by in
order to be accepted into the system.

✅ Disallow privileged containers

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  privileged: false
```

✅ Disallow sharing of the host process ID namespace ⚠️ **if `hostPID` is set to
`true` and a container is granted the `SYS_PTRACE` capability, it is possible to
escalate privileges outside the container**

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  hostPID: false
```

✅ Disallow sharing of the host IPC namespace (i.e. memory)

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  hostIPC: false
```

✅ Disallow sharing of the host network stack (i.e. access to loopback,
localhost, snooping on network traffic on local node)

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  hostNetwork: false
```

✅ Whitelist allowable volume types

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  # It's recommended to allow the core volume types.
  volumes:
    - "configMap"
    - "emptyDir"
    - "projected"
    - "secret"
    - "downwardAPI"
    # Assume that persistentVolumes set up by the cluster admin are safe to use.
    - "persistentVolumeClaim"
```

✅ Require containers to run as non-root user

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  runAsUser:
    # Require the container to run without root privileges.
    rule: "MustRunAsNonRoot"
  supplementalGroups:
    rule: "MustRunAs"
    ranges:
      # Forbid adding the root group.
      - min: 1
        max: 65535
```

✅ Set the `defaultAllowPrivilegeEscalation` to `false`

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  defautlAllowPrivilegeEscalation: false
```

✅ Apply Security Enhanced Linux (`seLinux`), `seccomp`, or `apparmor` profiles

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
  annotations:
    # applying default seccomp and apparmor profiles
    seccomp.security.alpha.kubernetes.io/allowedProfileNames: "docker/default"
    apparmor.security.beta.kubernetes.io/allowedProfileNames: "runtime/default"
    seccomp.security.alpha.kubernetes.io/defaultProfileName: "docker/default"
    apparmor.security.beta.kubernetes.io/defaultProfileName: "runtime/default"
```

### Full Pod Security Policy Example

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
  annotations:
    seccomp.security.alpha.kubernetes.io/allowedProfileNames: "docker/default"
    apparmor.security.beta.kubernetes.io/allowedProfileNames: "runtime/default"
    seccomp.security.alpha.kubernetes.io/defaultProfileName: "docker/default"
    apparmor.security.beta.kubernetes.io/defaultProfileName: "runtime/default"
spec:
  privileged: false
  # Required to prevent escalations to root.
  defautlAllowPrivilegeEscalation: false
  # This is redundant with non-root + disallow privilege escalation,
  # but we can provide it for defense in depth.
  requiredDropCapabilities:
    - ALL
  # Allow core volume types.
  volumes:
    - "configMap"
    - "emptyDir"
    - "projected"
    - "secret"
    - "downwardAPI"
    # Assume that persistentVolumes set up by the cluster admin are safe to use.
    - "persistentVolumeClaim"
  hostNetwork: false
  hostIPC: false
  hostPID: false
  runAsUser:
    # Require the container to run without root privileges.
    rule: "MustRunAsNonRoot"
  seLinux:
    # This policy assumes the nodes are using AppArmor rather than SELinux.
    rule: "RunAsAny"
  supplementalGroups:
    rule: "MustRunAs"
    ranges:
      # Forbid adding the root group.
      - min: 1
        max: 65535
  fsGroup:
    rule: "MustRunAs"
    ranges:
      # Forbid adding the root group.
      - min: 1
        max: 65535
  readOnlyRootFilesystem: false
```

---

## Network Policies

Kubernetes allows users to deploy their choice of network add-ons.

✅ Choose a network add-on that allows you to leverage Network Policies, like
[Calico](https://docs.projectcalico.org/v3.1/introduction/) or
[Canal](https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/flannel),
which gives you networking via Flannel and network policies via Calico.

```sh
# Canal Example
$ kube-apiserver                 \
    <... other flags>            \
    --cluster-cidr=10.244.0.0/16 \
    --allocate-node-cidrs=true

# install RBAC
$ kubectl apply -f \
    https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/canal/rbac.yaml

# install Calico
$ kubectl apply -f \
    https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/canal/canal.yaml
```

## Additional Security Measures

All recommendations up to this point are just to get you up and running. In this
section, I am going to cover some additional security measures!

✅ Kubernetes
[Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) are useful
for some limited use-cases. I wouldn't rely on it as a secrets management
solution. Instead, consider Hashicorp [Vault](https://www.vaultproject.io/) for
that.

✅ Continuously scan for security vulnerabilities in your containers with
[Anchore](https://anchore.com/) or
[Clair](https://coreos.com/clair/docs/latest/).

✅ Keep your infrastructure up-to-date on security patches, or run Kubernetes on
OSes that keep themselves up-to-date (e.g. CoreOS)

✅ Only deploy authorized container images that you've analyzed, scanned, and
signed (i.e. Software Supply Chain Security). [Grafeas](https://grafeas.io/),
[TUF](https://github.com/theupdateframework/tuf), and
[Notary](https://github.com/theupdateframework/notary) can help here.

✅ Limit direct access to the Kubernetes nodes.

✅ Avoid noisy neighbor problems. Define resource quotas.

✅ Monitor and log everything with Prometheus and Grafana.
[Sysdig Falco](https://github.com/draios/falco) will detect and alert on
anomalous container behavior, like shell execution in a container, container
privilege escalation, spawning of unexpected child processes, mounting sensitive
paths, system binaries making network connections ...etc

---

## Additional References

- [Securing a Cluster](https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/)
- [Security Best Practices for Kubernetes Deployments](https://kubernetes.io/blog/2016/08/security-best-practices-kubernetes-deployment/)
- [Kubernetes Security Best-Practice](https://github.com/freach/kubernetes-security-best-practice)

---

## Conclusion

Containers and Orchestrators are not inherently more or less secure than
traditional virtualization technologies. If anything, I personally think
containers and orchestrators have the potential to revolutionize the security
industry and truly enable _Security at the speed of DevOps_.

If you feel like I missed something, got some details wrong, or just want to say
hi, please feel free to leave a comment below or reach out to me on
[GitHub 🐙](https://github.com/petermbenjamin/ama),
[Twitter 🐦](https://twitter.com/petermbenjamin), or
[LinkedIn 🔗](https://www.linkedin.com/in/pmbenjamin)
