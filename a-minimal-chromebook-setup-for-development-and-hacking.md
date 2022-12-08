---
date: 2018-05-06
tags: ["chromebook", "development", "hacking", "minimal"]
categories: ["development"]
---

# A Minimal Chromebook Setup for Development and Hacking

## Table of Contents

- [A Minimal Chromebook Setup for Development and Hacking](#a-minimal-chromebook-setup-for-development-and-hacking)
  - [Table of Contents](#table-of-contents)
  - [The TLDR Version](#the-tldr-version)
  - [Introduction](#introduction)
  - [The SOLID Principles](#the-solid-principles)
    - [Security](#security)
    - [Ownership Cost](#ownership-cost)
    - [Leverage Native Capabilities](#leverage-native-capabilities)
    - [Inherent Restrictions](#inherent-restrictions)
    - [Developer Experience](#developer-experience)
  - [The Details](#the-details)
    - [Hardware](#hardware)
    - [Software](#software)
  - [Parting Thoughts](#parting-thoughts)

## The TLDR Version

- If you buy a Chromebook, stay within the confines and stick with ChromeOS;
  it’s not worth hacking to enable dual-booting, side-loading, or wiping of
  ChromeOS for a Linux distro.
  - If you insist on a Linux distro, just buy a used Lenovo; it will be more
    capable, versatile, and compatible with your Linux distro out-of-the-box.
  - Whatever you do, **DO NOT** enable "Dev Mode".
- Best-in-class Chromebook hardware ≤
  $500 (in order):
  [Samsung Chromebook Pro](https://www.samsung.com/us/computing/chromebooks/12-14/samsung-chromebook-pro-xe510c24-k01us/)
  (approx. $500),
  [Asus Chromebook Flip C302](https://www.asus.com/us/2-in-1-PCs/ASUS-Chromebook-Flip-C302CA/)
  (approx.
  $450),
  [Acer Chromebook for Work](https://www.amazon.com/Acer-ChromeBook-CP5-471-35T4-Black-NX-GE8AA-002/dp/B01EPZIJRQ/ref=sr_1_4?ie=UTF8&qid=1523224085&sr=8-4&keywords=chromebook+for+work)
  (approx. $400).
  - Otherwise, aim for an Intel-based chip (e.g. Celeron) and a minimum of 4 GB
    RAM.
- [Secure Shell](https://chrome.google.com/webstore/detail/secure-shell-app/pnhechapfaindjhompbnflcldabbghjo):
  the Chrome extension for SSH. Multi-Factor Authentication (MFA) highly
  recommended.
- [Termux](https://termux.com/): a full-featured terminal-emulator Android-app
  with lots of developer-friendly plugins, like access to clipboard,
  notifications, external storage, and more.
- GUI-based editors are also available as Chrome Apps, like
  [Caret](https://chrome.google.com/webstore/detail/caret/fljalecfjciodhpcledpamjachpmelml?hl=en),
  [Zed](https://chrome.google.com/webstore/detail/zed-code-editor/pfmjnmeipppmcebplngmhfkleiinphhp?hl=en),
  &
  [Text](https://chrome.google.com/webstore/detail/text/mmfbcljfglbokpmkimbfghdkjmjhdgbg?hl=en).
  - [Chrome Dev Editor](https://github.com/googlearchive/chromedeveditor) was a
    really nice IDE that almost matched modern desktop-based editors on features
    (e.g. Atom & VS Code), but it is not longer in active development,
    unfortunately.

---

## Introduction

I decided to share my recommended Chromebook development setup because I am
tired of
[seeing](https://www.theverge.com/2017/11/16/16656420/google-pixelbook-chromebook-development-linux-crouton-how-to)
[the](http://programmingzen.com/developing-with-a-chromebook/)
[same](https://arstechnica.com/gadgets/2017/06/how-to-install-linux-on-a-chromebook/)
[misinformed](https://headmelted.com/coding-on-a-chromebook-84335cce96c8)
[and](https://medium.com/@martinmalinda/ultimate-guide-for-web-development-on-chromebook-part-1-crouton-2ec2e6bb2a2d)
[misguided](https://gist.github.com/rachelmyers/d7023ef34e58fe925f9c)
recommendations that lead to:

- A painful development experience
- A cheap Linux machine with terrible driver support
- Wildly insecure and highly risky machines

So, how will my recommendation be different than those previously listed?

I have my own SOLID set of guiding principles that I will discuss in detail.

_Note:_ Keep in mind that there is no setup without its own set of drawbacks.

## The SOLID Principles

### Security

ChromeOS comes with a set of really — and I mean REALLY — strong security
controls built-in. So much so that CoreOS, a specialized server operating system
for containers, is based on ChromeOS.

So, it is justifiably cringe-worthy that the first recommendation would be
“disable all those secure features and turn on Dev Mode”.

Why is Dev Mode insecure?

- Disables Verified Boot.
  - Verified Boot ensures that ChromeOS is booting using known-good firmware,
    kernel, init, modules, fs metadata, policies, …etc.
  - By disabling Verified Boot, you’re essentially allowing persistent
    compromises.
- Enables VT2 (Linux terminal).
- Activates passwordless root shell access. Yes, you read that right.
- Access to unencrypted content of your Chrome profile.

On top of that, adding Crouton to side-load Linux distributions adds another
huge vector. You’re essentially running un-vetted code as root on your
Chromebook. Not only that, but nothing is stopping code running as root in a
chrooted environment from escaping the chroot and infecting the rest of
ChromeOS.

> Essentially, dev mode by default is less physically secure than a standard
> laptop running Linux -
> [David Schneider](https://github.com/dnschneid/crouton/wiki/Security) (creator
> of Crouton)

If you insist on a Linux distribution, you are better served by finding an old,
used, cheap Lenovo Thinkpad and installing your favorite Linux distribution on
it. You’ll have a more secure environment than a Chromebook in Dev Mode and a
better overall experience than a Chromebook running a Linux distribution.

### Ownership Cost

- Avoid any recommendations that would unnecessarily raise the cost of ownership
  (e.g. buying additional hardware/software to perform needed tasks).
- Keep costs as low as possible.

### Leverage Native Capabilities

- Leverage the native capabilities of the platform (e.g. web apps > android
  apps > containers).
- Avoid unnecessary hacks that impact performance.

### Inherent Restrictions

- The Chromebook stack (hardware & software) is optimized for ChromeOS.
  - Thus, you should accept and embrace the inherent restrictions that come with
    the decision of using a Chromebook. They ultimately force you treat your
    machine as cattle, not pets.
  - Your profile, preferences, settings, files are stored in the cloud. If you
    lose your Chromebook, you can just replace it and keep going with little
    disruption and inconvenience.
- Attempts at bypassing or mitigating these restrictions will result in
  compromises that will impact one or more aspects of the stack (incompatible
  drivers, slow/weak performance, poor battery life, lack of security …etc).

### Developer Experience

- Strive for the smoothest developer experience possible.
- I define Developer experience as, but not limited to:
  - battery life
  - performance
  - productivity
  - tooling
  - flexibility/versatility of working offline

## The Details

I have been using a Chromebook as my primary personal machine for a few years
now. So, I have experimented with a variety of different hardware and software
combinations. I am also very familiar with the initial pains of getting
acclimated to this new setup. But, once I’ve adapted to it, I noticed that I
have applied the same philosophy to my work setup.

I also consider myself a pro user with often conflicting needs:

On the one hand, as a Software Engineer, I prefer to work on my personal and
work-related projects on remote development environments (I would also like to
be able to do local/offline development when absolutely needed). On the other
hand, as an Information Security Engineer, SSH'ing into a remote server is not
an option and I certainly need to be able to — erm, do some things 😏 — directly
from my local machine.

Here is where I have landed…

### Hardware

- Prefer Intel-based chips over ARM ones.
  - Most developer programs & tools are built for and supported on Intel chips
    first.
- Prefer larger RAM.
  - A minimum of 4GB RAM.
  - I found that 2GB is barely usable, especially with the recently added
    support for Android apps.
- Prefer larger storage.
  - 64GB storage is nice, but 32GB will work too.
  - You need to take into account how much local development versus remote
    development you are comfortable with.

Based on these loose hardware recommendations, the following are the top 3:

- Asus Chromebook Flip C302 (~\$450)
  - Intel m3
  - 4GB RAM
  - 64GB storage.
- Samsung Chromebook Pro (~\$500)
  - Intel m3
  - 4GB RAM
  - 32GB storage
  - Stylus included
- Acer Chromebook for Work (~\$400)
  - Intel 6th-gen i3
  - 4GB RAM
  - 32GB storage.

Alternatively, these are significantly more affordable and could be sufficient
for your use-case:

- Samsung Chromebook 3 (approx. \$200)
- 2017 Samsung Chromebook 11.6inch (approx. \$250)
- ASUS C300 (approx. \$200)

### Software

I will be straight-forward with you here: **get comfortable living in the
terminal**.

GUI-based IDEs/editors for local software development on Chromebooks are not
there yet, but they are getting better.

There are some lightweight GUI editors, but they lack many critical features
needed for software development (e.g. git integration, search-and-replace,
auto-completion …etc). These lightweight GUI editors are good for light
editing/note-taking at best.

This is a good thing and I'll tell you why:

The Kubernetes and Docker container movement has ushered in an era of DevOps
that can be summarized as "_Pets versus Cattle_":

> In the old way of doing things, we treat our servers like pets, for example
> Bob the mail server. If Bob goes down, it’s all hands on deck. The CEO can’t
> get his email and it’s the end of the world. In the new way, servers are
> numbered, like cattle in a herd. For example, www001 to www100. When one
> server goes down, it’s taken out back, shot, and replaced on the line. -
> [Randy Bias](http://cloudscaling.com/blog/cloud-computing/the-history-of-pets-vs-cattle/)

Here is the baseline software stack I use for development (all of which are
installed on the server):

- Secure Shell: a Chrome extension, set to “open as window”, is my go-to SSH
  client to remote into remote development environments (public cloud for work,
  private home servers for personal projects) \* **Public Service
  Announcement:** since installing client-side certificates on Chromebooks are
  excruciatingly difficult, I resort to password-based SSH authentication for my
  servers. But if you do this, please do yourself a favor and enable
  Multi-Factor Authentication (MFA) for SSH
  ([DUO](https://github.com/petermbenjamin/dotfiles/blob/master/duo/duo-pam.sh)
  is great!)
- Tmux: terminal window and pane management
- Vim: editor
- Docker: further compartmentalized dev environments (e.g. `docker-compose up`
  and I've got a local NGINX server, Ruby app server, & postgresql server up and
  running)

Here is the baseline software stack I use for hacking:

- Same as above, plus
- Termux: android-based terminal emulator for performing local testing when
  needed.

And that's it. I have reached a point where all I need is a phone, a portable
monitor, and a keyboard to be productive.

And that's what this journey has been really all about: bringing technology back
to its most basic set of needs for me to be effective and productive, where I am
no longer physically or emotionally dependent on premium brushed aluminum
computers with 16GB RAM just to run slack, chrome, and atom or vscode.

## Parting Thoughts

I don't claim that this setup is for everyone. Admittedly, this setup falls
apart when you have a hard requirement on a piece of software that is not
available as a web app or if it is financially prohibitive as a web app.

I am really excited about what the future of development might look like.

Perhaps, in the not-so-distant-future, all you might need is the multi-core
processing power already in your pocket, a Virtual Reality (VR) or Augmented
Reality (AR) headset, with virtual workspaces (because, why be prohibitively
limited by physical, costly monitors when you can just create unlimited number
virtual monitors?)

But, I'm curious about your thoughts, opinions, or concerns about this setup or
if you would like to share your setup.

Thank you for your time.

Cheers and happy coding! 🤗
