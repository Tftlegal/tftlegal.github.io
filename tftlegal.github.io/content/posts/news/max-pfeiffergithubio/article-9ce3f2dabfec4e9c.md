---
title: "Overhauling my Ceph cluster (again)"
date: 2026-02-15T16:30:00Z
source: "max-pfeiffergithubio"
original_url: "https://max-pfeiffer.github.io/overhauling-my-ceph-cluster-again.html"
summary: "The text is a brief title or heading indicating a hardware change. It describes the removal of Waveshare PoE (Power over Ethernet) HATs from a project. The author is transitioning away from PoE-based power delivery. Instead, USB power supplies will be used to power the devices. This suggests a simplification or cost reduction of the power infrastructure. The change implies that Ethernet-based power delivery is no longer needed or practical for the setup."
---

# Overhauling my Ceph cluster (again)

## Краткое содержание

The text is a brief title or heading indicating a hardware change. It describes the removal of Waveshare PoE (Power over Ethernet) HATs from a project. The author is transitioning away from PoE-based power delivery. Instead, USB power supplies will be used to power the devices. This suggests a simplification or cost reduction of the power infrastructure. The change implies that Ethernet-based power delivery is no longer needed or practical for the setup.

## Полная статья

Wed 01 October 2025

I already knew it when I was powering up my Ceph cluster the first time: the fans on that[Waveshare POE HATs (F)](https://www.waveshare.com/poe-hat-f.htm)will be a problem at some point in the future. As there is no fan speed control in place, they are running at full throttle all the time. After running the cluster 24/7 for 10 months, the wear and tear on that fans caused a big increase in fan noise. And the Ceph cluster is placed right next to my work place. That fan noise became worse and worse and was so annoying at some point that I decided to swap out the[Waveshare POE HATs (F)](https://www.waveshare.com/poe-hat-f.htm).

So I was looking at alternative POE HATs without fans first, and I found a couple on the market now. For instance, the[Waveshare POE HAT H](https://www.waveshare.com/product/raspberry-pi/hats/interface-power/poe-hat-h.htm). But these are more on the pricy side. And browsing through all the new stuff on Waveshare website, I found another interesting[HAT providing two 2.5GB ethernet ports and a NVMe SSD mount](https://www.waveshare.com/product/raspberry-pi/hats/interface-power/pcie-to-m.2-usb-eth-hat-plus.htm). This way I could address the network throughput bottleneck I am currently facing as well. The downside of using this HAT would be to put the operating system on the SD-Card again, which is not optimal.

I decided to just go for USB power supplies eventually as this was the cheapest and least invasive solution I could do at this point. I already had one Raspberry Pi 5 power supply unit and an active cooler on my shelf. So I just had to buy two more. Trying to place all three power supply units in my rack mounted socket strip, I noticed that these guys need their space. So it was not possible to put all three into that socket strip because they were interfering with each other. So I just bought another socket strip for my rack. So I ended up with this part list:

- 3x[Raspberry Pi 5 power supply](https://www.raspberrypi.com/products/27w-power-supply/)
- 3x[Raspberry Pi 5 active cooler](https://www.raspberrypi.com/products/active-cooler/)
- 1x 10'' rack mounted socket strip

![2025-10-01_overhauling_my_ceph_cluster_again_parts.jpeg](/news/max-pfeiffergithubio/article-9ce3f2dabfec4e9c/image-01.jpeg)

## Shutting down the Ceph cluster

As a first step, I had to shut down my Ceph cluster. As I need to shut it down completely, data availability will be impacted. So I had to do make the consumers digest that first. You want to gracefully shut it down. Therefore, you need to stop all running services before you shut down the machines:

```
cephadm shell
ceph orch stop --all
```

In practice, I noticed that a Ceph cluster is a very resilient thing and can take quite some beating. So just shutting down single machines or all of them did no harm in my experience. I also had complete power outages, and the Ceph cluster came up fine again.

## Installation

First I had to get rid of the bad boys: I unscrewed the Raspberry Pis and took out the noisy POE HATs.

![2025-10-01_overhauling_my_ceph_cluster_again_old_poe_hat.jpeg](/news/max-pfeiffergithubio/article-9ce3f2dabfec4e9c/image-02.jpeg)

Then I installed the active cooler on all devices and put it back together.

![2025-10-01_overhauling_my_ceph_cluster_again_installation_done.jpeg](/news/max-pfeiffergithubio/article-9ce3f2dabfec4e9c/image-03.jpeg)

Lastly, I put the socket strip into the 10'' rack together with the power supply units. After powering on all devices I was checking the Ceph cluster's dashboard and was happy to see that everything was running smoothly again. And I noticed something crucial ... silence!

![2025-10-01_overhauling_my_ceph_cluster_again_result.jpeg](/news/max-pfeiffergithubio/article-9ce3f2dabfec4e9c/image-04.jpeg)

## Outlook

I am not sure if I will tinker around with the Raspberrys more than I did already. There are some interesting upgrade options that I mentioned earlier. But after looking closer at some other single board computers out there, I stumbled across the new[Banana PI BPI-M7](https://www.banana-pi.org/en/banana-pi-sbcs/169.html). It already comes with all the desired features: two 2.5GB ethernet ports, NVMe SSD adapter and more memory. This comes with a price tag of 220 Euro for the 16GB variant at AliExpress currently. So I rather might get one of these and try to grow my Ceph cluster with an additional Banana Pi node.

Another interesting option is the[UGREEN NASync DXP2800](https://nas-eu.ugreen.com/products/ugreen-nasync-dxp2800-nas-storage)NAS. It comes with an Intel N100 Quad-core CPU, 8GB DDR5 RAM and a 2.5Gb ethernet port. It can carry two NVMe SSDs internally and two more SATA drives in its two drive bays. I saw these ranging around 290 Euro. On these machines you can[install Debian on its eMMC flash memory drive](https://wiki.debian.org/InstallingDebianOn/Ugreen). And then add your Ceph installation ontop of it.

## Related Articles

- [Ceph Cluster with Raspberry Pi 5 and NVMe SSDs](https://max-pfeiffer.github.io/ceph-cluster-with-raspberry-pi-5-and-nvme-ssds.html)
- [Configuring and using Ceph CSI Driver for Kubernetes](https://max-pfeiffer.github.io/configuring-and-using-ceph-csi-driver-for-kubernetes.html)
- [Overhauling my Ceph cluster](https://max-pfeiffer.github.io/overhauling-my-ceph-cluster.html)

## Оригинал

https://max-pfeiffer.github.io/overhauling-my-ceph-cluster-again.html
