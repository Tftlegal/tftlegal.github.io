---
title: "Provisioning a Kubernetes Cluster with Talos Linux and Proxmox VE with OpenTofu"
date: 2026-02-10T17:19:00Z
source: "max-pfeiffergithubio"
original_url: "https://max-pfeiffer.github.io/provisioning-a-kubernetes-cluster-with-talos-linux-and-proxmox-ve-with-opentofu.html"
summary: "Проект представляет собой proof of concept (доказательство концепции) по развёртыванию инфраструктуры. Его цель — автоматизированное provisioning кластера Kubernetes. В качестве операционной системы для узлов используется Talos Linux — минималистичная, Kubernetes-ориентированная ОС. Виртуализация обеспечивается гипервизором Proxmox VE. Для управления инфраструктурой как кодом (IaC) применяется OpenTofu — открытый форк Terraform. Таким образом, проект объединяет три ключевых компонента: Kubernetes, Talos Linux и Proxmox VE, связывая их в единый pipeline на базе OpenTofu. Основной акцент сделан на проверке работоспособности данного стека, а не на промышленное развёртывание."
---

# Provisioning a Kubernetes Cluster with Talos Linux and Proxmox VE with OpenTofu

## Краткое содержание

Проект представляет собой proof of concept (доказательство концепции) по развёртыванию инфраструктуры. Его цель — автоматизированное provisioning кластера Kubernetes. В качестве операционной системы для узлов используется Talos Linux — минималистичная, Kubernetes-ориентированная ОС. Виртуализация обеспечивается гипервизором Proxmox VE. Для управления инфраструктурой как кодом (IaC) применяется OpenTofu — открытый форк Terraform. Таким образом, проект объединяет три ключевых компонента: Kubernetes, Talos Linux и Proxmox VE, связывая их в единый pipeline на базе OpenTofu. Основной акцент сделан на проверке работоспособности данного стека, а не на промышленное развёртывание.

## Полная статья

Wed 25 December 2024

The whole story started for me when a colleague recommended[Talos Linux](https://www.talos.dev/)for building a Kubernetes cluster on bare metal some weeks ago. He sent me a video from[DHCP 2024 conference](https://dhcp.cfhn.it/)which dragged me into that topic (sorry for the German). I was amazed by that concept and immediately started to inhale the docs and related projects on GitHub.

[Talos Linux](https://www.talos.dev/)is a special Linux distribution geared towards its only purpose which is providing a platform for Kubernetes. It comes with an API which can be utilized to provision and configure the OS installation and the Kubernetes cluster. That makes it an ideal partner for provisioning Tools like[OpenTofu](https://opentofu.org/)or Terraform. Consequently, there is no direct shell access to the Talos machines. The company behind[Talos Linux](https://www.talos.dev/)is[Siderolabs](https://www.siderolabs.com/).

There is this official[terraform-provider-talos](https://github.com/siderolabs/terraform-provider-talos)from[Siderolabs](https://www.siderolabs.com/)that you can use.

I like that infrastructure-as-code approach, so I started a proof of concept project with[OpenTofu](https://opentofu.org/)the next weekend. I already have Proxmox VE as hypervisor running in my home lab, so I spun up some virtual machines on that server using the[Talos Linux](https://www.talos.dev/)iso image and used the[Talos Terraform provider](https://github.com/siderolabs/terraform-provider-talos)to provision the Kubernetes cluster.

I was absolutely amazed how easy that was. I just spent roughly three hours to put together this little POC project:[https://github.com/max-pfeiffer/proxmox-talos-opentofu](https://github.com/max-pfeiffer/proxmox-talos-opentofu)

I encourage everyone to give Talos Linux a shot and try it out.

## Оригинал

https://max-pfeiffer.github.io/provisioning-a-kubernetes-cluster-with-talos-linux-and-proxmox-ve-with-opentofu.html
