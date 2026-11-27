---
title: "New Release: Tails 7.15.1"
date: 2026-10-23T00:00:00Z
source: "blogtorprojectorg"
original_url: "https://blog.torproject.org/new-release-tails-7_15_1/"
summary: ""
---

# New Release: Tails 7.15.1

## Краткое содержание



## Полная статья

This release is an emergency release to remove a compromised certificate authority.

## Changes and updates

- 

Update*Tor Browser*to[15.0.26](https://blog.torproject.org/new-release-tor-browser-15026/), which fixes[CVE-2026-113470](https://app.opencve.io/cve/CVE-2026-113470).

On[October 17](https://www.mozilla.org/en-US/security/advisories/mfsa2026-112/),*Mozilla*blocked the certificates of[eMudhra, a certificate authority that had its intermediate certificates compromised](https://en.wikipedia.org/wiki/EMudhra).*Tor Browser*15.0.26 now blocks all eMudhra certificates as well.

Using one of the compromised intermediate certificates, an attacker could impersonate any website, even if the connection is secured with HTTPS.

This attack could be performed by a Tor exit node, the Internet service provider of an exit node, or a government. Onion sites are not vulnerable.

We are not aware of this attack being used by exit nodes so far.

- 

Update the certificate authorities to block all eMudhra certificates in other applications.

In addition to*Tor Browser*, other applications rely on the certificate authorities included in Tails with the`ca-certificates`Debian package to secure their connections.

## Get Tails 7.15.1

### To upgrade your Tails USB stick and keep your Persistent Storage

- 

Automatic upgrades are available from Tails 7.0 or later to 7.15.1.

- 

If you cannot do an automatic upgrade or if Tails fails to start after an automatic upgrade, please try to do a[manual upgrade](https://tails.net/doc/upgrade/#manual).

### To install Tails 7.15.1 on a new USB stick

Follow our[installation instructions](https://tails.net/install/).

The Persistent Storage on the USB stick will be lost if you install instead of upgrading.

### To download only

If you don't need installation or upgrade instructions, you can download Tails 7.15.1 directly:

- 

[For USB sticks (USB image)](https://tails.net/install/download/)

- 

[For DVDs and virtual machines (ISO image)](https://tails.net/install/download-iso/)

## Support and feedback

For support and feedback, visit the[Support section](https://tails.net/support/)on the Tails website.

## Оригинал

https://blog.torproject.org/new-release-tails-7_15_1/
