+++
date = '2023-11-18T00:00:00+02:00'
draft = false
title = 'Reviving an Old Office PC into an UNRAID Server'
description = 'Transforming an old office PC into an Unraid server — hardware upgrades, setup process, and usage tips.'
categories = ['Projects', 'Homelab']
tags = ['Docker', 'Unraid', 'Server', 'DIY']
+++
{{< lead >}}
And diving into self-hosting.
{{< /lead >}}
After visiting a definitely not shady gumtree seller, I managed to acquire an ex office Dell workstation packed with the latest and greatest such as:
- i7 6700 (non k 😔)
- 8GB of Ram
- A 500GB HDD

A further marketplace purchase for 32GB of DDR4, plus shucking some Seagate HDDs for storage, it was finally ready to go.

Since then, the server has seen many different upgrades such as a hand me down *Threadripper 1950X*, even more RAM bringing it up to 64GB, and of course always more storage.

It has been an amazing journey of learning and I can recommmend UNRAID OS to anyone, as 'It just works.':tm: Well, mostly - I did have one issue when the USB which the OS ran on started to degrade which initially seemed like a problem with one of my new hard drives or the SATA ports and cables, but all is forgiven after a new USB stick.

Some of my self-hosted services include:

- **kidami.xyz** – ~~you're here~~ you were here! I initially hosted my first overly complicated site on a VM on the server, but decided to make something newer and simpler. You can read more about this [here]({{< relref "../old_kidami.md" >}})
- Initially a photo sharing site **Piwigo** which later changed to a photo sharing and backup solution **Immich**.
- A media server.
- Multiple game servers.
- File sharing and backup services (**Syncthing, Restic** via **Backrest**)
- Utilities like an **InfluxDB, Telegraf, and Grafana** stack for system monitoring, and recently a pretty cool dashboard for my **Garmin** watch stats.

Most of these run on **Docker**, thanks to UNRAID's extensive community apps library. The main exception is my old website, which ran on its own virtual machine.

This is a never-ending project, as every now and then I find yet another interesting service I want to try self-hosting.