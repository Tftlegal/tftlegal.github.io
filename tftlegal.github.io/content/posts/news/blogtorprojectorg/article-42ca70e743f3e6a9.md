---
title: "Arti 2.8.0 released"
date: 2026-11-03T00:00:00Z
source: "blogtorprojectorg"
original_url: "https://blog.torproject.org/arti_2_8_0_released/"
summary: ""
---

# Arti 2.8.0 released

## Краткое содержание



## Полная статья

Arti is our ongoing project to create a next-generation Tor implementation in Rust. We're happy to announce the latest release, Arti 2.8.0.

This release continues our ongoing development towards using Arti as a relay and as a directory authority, with much of the work focusing on computing the consensus from votes, bootstrapping router descriptors and microdescriptors, as well as relay bandwidth rate-limiting, descriptor encoding, and the relay DNS cache implementation. Additionally, we have made a number of cleanups and clarifications to the Tor specification.

This release contains security fixes for a number of low-severity security issues. We encourage`arti`and`arti-client`users, especially those running onion services, to upgrade as soon as possible.

As usual, there also are many under-the-hood improvements to our infrastructure, testing, and documentation, along with multiple bug fixes and internal cleanups.

For full details on what we've done, including API changes, security fixes, and for information about many more minor and less-visible changes, please see the[CHANGELOG](https://gitlab.torproject.org/tpo/core/arti/-/blob/main/CHANGELOG.md#arti-280--3-november-2026).

For more information on using Arti, see our top-level[README](https://gitlab.torproject.org/tpo/core/arti/-/blob/main/README.md), and the documentation for the[`arti`binary](https://gitlab.torproject.org/tpo/core/arti/-/blob/main/crates/arti/README.md).

Thanks to everybody who's contributed to this release, including 5225225, Gaith Hallak, and manthan-lab.

Also, our deep thanks to our[sponsors](https://www.torproject.org/about/sponsors/)for funding the development of Arti!

## Оригинал

https://blog.torproject.org/arti_2_8_0_released/
