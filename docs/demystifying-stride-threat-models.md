---
date: 2018-08-18
tags: ["security", "threat models", "stride", "application security"]
---

# Demystifying Stride Threat Models

![stride](../img/stride.jpg)

## Table of Contents

- [Demystifying Stride Threat Models](#demystifying-stride-threat-models)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [What is a Threat Model?](#what-is-a-threat-model)
  - [What is STRIDE?](#what-is-stride)
    - [Spoofing](#spoofing)
    - [Tampering](#tampering)
    - [Repudiation](#repudiation)
    - [Information Disclosure](#information-disclosure)
    - [Denial of Service](#denial-of-service)
    - [Elevation of Privileges](#elevation-of-privileges)
  - [Summary](#summary)
  - [Additional Resources](#additional-resources)
  - [Footnotes](#footnotes)
  - [Updates](#updates)

## Introduction

Software is eating the world. As a result, the repercussions of software failure
is costly and, at times, can be catastrophic. This can be seen today in a wide
variety of incidents, from data leak incidents caused by misconfigured
[AWS S3 buckets](https://github.com/petermbenjamin/yas3bl) to Facebook data
breach incidents due to
[lax API limitations](https://techcrunch.com/2018/06/28/facepalm-2/) to the
Equifax incident due to the use of
[an old Apache Struts version with a known critical vulnerability](https://www.synopsys.com/blogs/software-security/equifax-apache-struts-cve-2017-5638-vulnerability/).

Application Security advocates encourage developers and engineers to adopt
security practices as early in the Software Development Life Cycle (SDLC) as
possible <sup>[1](#tanya-janca)</sup>. One such security practice is **Threat
Modeling**.

In this article, I offer a high-level introduction to one methodology, called
STRIDE, and in a future article, I will demonstrate this process using an
existing open-source application as an example.

## What is a Threat Model?

Here is the obligatory
[Wikipedia definition](https://en.wikipedia.org/wiki/Threat_model):

> Threat modeling is a process by which potential threats, such as structural
> vulnerabilities, can be identified, enumerated, and prioritized – all from a
> hypothetical attacker’s point of view. The purpose of threat modeling is to
> provide defenders with a systematic analysis of the probable attacker’s
> profile, the most likely attack vectors, and the assets most desired by an
> attacker.

With that out of the way, the simplest explanation in English is this:

_Threat Models are a systematic and structured way to identify and mitigate
security risks in our software_.

There are various ways and methodologies of doing threat models, one of which is
a process popularized by Microsoft, called _STRIDE_.

## What is STRIDE?

STRIDE is an acronym that stands for 6 categories of security risks: Spoofing,
Tampering, Repudiation, Information Disclosure, Denial of Service, and Elevation
of Privileges.

Each category of risk aims to address one aspect of security.

Let's dive into each of these categories.

### Spoofing

Spoofing refers to the act of posing as someone else (i.e. spoofing a user) or
claiming a false identity (i.e. spoofing a process).

This category is concerned with **authenticity**.

Examples:

- One user spoofs the identify of another user by brute-forcing
  username/password credentials.
- A malicious, phishing host is set up in an attempt to trick users into
  divulging their credentials.

You would typically mitigate these risks with proper
[authentication](https://www.owasp.org/index.php/Authentication_Cheat_Sheet).

### Tampering

Tampering refers to malicious modification of data or processes. Tampering may
occur on data in transit, on data at rest, or on processes.

This category is concerned with **integrity**.

Examples:

- A user performs
  [bit-flipping attacks](https://en.wikipedia.org/wiki/Bit-flipping_attack) on
  data in transit.
- A user modifies data at rest/on disk.
- A user performs
  [injection attacks](https://en.wikipedia.org/wiki/Code_injection) on the
  application.

You would typically mitigate these risks with:

- Proper validation of users' inputs and proper encoding of outputs.
- Use prepared SQL statements or stored procedures to mitigate SQL injections.
- Integrate with security static code analysis tools to identify security bugs.
- Integrate with composition analysis tools (e.g. `snyk`, `npm audit`, BlackDuck
  ...etc) to identify 3rd party libraries/dependencies with known security
  vulnerabilities.

### Repudiation

Repudiation refers to the ability of denying that an action or an event has
occurred.

This category is concerned with **non-repudiation**.

Examples:

- A user denies performing a destructive action (e.g. deleting all records from
  a database).
- Attackers commonly erase or truncate log files as a technique for hiding their
  tracks.
- Administrators unable to determine if a container has started to behave
  suspiciously/erratically.

You would typically mitigate these risks with proper
[audit logging](https://www.computerweekly.com/tip/Best-practices-for-audit-log-review-for-IT-security-investigations).

### Information Disclosure

Information Disclosure refers to data leaks or data breaches. This could occur
on data in transit, data at rest, or even to a process.

This category is concerned with **confidentiality**.

Examples:

- A user is able to eavesdrop, sniff, or read traffic in clear-text.
- A user is able to read data on disk in clear-text.
- A user attacks an application protected by TLS but is able to steal x.509
  (SSL/TLS certificate) decryption keys and other sensitive information.
  [Yes, this happened](https://en.wikipedia.org/wiki/Heartbleed).
- A user is able to read sensitive data in a database.

You would typically mitigate these risks by:

- Implementing proper
  [encryption](https://www.owasp.org/index.php/Cryptographic_Storage_Cheat_Sheet).
- Avoiding self-signed certificates. Use a valid, trusted Certificate Authority
  (CA).

### Denial of Service

Denial of Service refers to causing a service or a network resource to be
unavailable to its intended users.

This category is concerned with **availability**.

Examples:

- A user performs [SYN flood](https://en.wikipedia.org/wiki/SYN_flood) attack.
- The storage (i.e. disk, drive) becomes too full.
- A Kubernetes dashboard is left exposed on the Internet, allowing anyone to
  deploy containers on your company's infrastructure to mine cryptocurrency and
  starve your legitimate applications of CPU.
  [Yes, that happened too](https://redlock.io/blog/cryptojacking-tesla).

Mitigating this class of security risks is tricky because solutions are highly
dependent on a lot of factors.

For the Kubernetes example, you would mitigate resource consumption with
[resource quotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/).

For a storage example, you would mitigate this with proper log rotation and
monitoring/alerting when disk is nearing capacity.

### Elevation of Privileges

Elevation of Privileges refers to gaining access that one should not have.

This category is concerned with **authorization**.

Example:

- A user takes advantage of a Buffer Overflow to gain root-level privileges on a
  system.
- A user with limited to no permissions to Kubernetes can elevate their
  privileges by sending a specially crafted request to a container with the
  Kubernetes API server's TLS credentials.
  [Yes, this was possible](https://github.com/kubernetes/kubernetes/issues/71411).

Mitigating these risks would require a few things:

- Proper authorization mechanism (e.g. role-based access control).
- Security static code analysis to ensure your code has little to no security
  bugs.
- Compositional analysis (aka dependency checking/scanning), like
  [`snyk`](https://snyk.io) or [`npm audit`](https://docs.npmjs.com/cli/audit),
  to ensure that you're not relying on known-vulnerable 3rd party dependencies.
- Generally practicing least privilege principle, like running your web server
  as a non-root user.

## Summary

So, STRIDE is a threat model methodology that should help you systematically
examine and address gaps in the security posture of your applications.

In a future article, we'll take an application and go through this process so
you can get a feel for how this works.

If you would like to propose an application for me to threat model next, feel
free to drop suggestions in the comments below.

## Additional Resources

- https://en.wikipedia.org/wiki/STRIDE_(security)
- https://www.webtrends.com/blog/2015/04/threat-modeling-with-stride/
- https://www.oreilly.com/library/view/threat-modeling-designing/9781118810057/9781118810057c03.xhtml

## Footnotes

- <a name="tanya-janca">1</a>:
  https://www.youtube.com/watch?v=YGJqpQy79no&WT.mc_id=shehackspurple-blog-tajanca

## Updates

1. Add more mitigations against tampering.
2. Add more mitigations against information disclosure.
