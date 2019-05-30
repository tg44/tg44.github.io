---
title: Current home-server setup
layout: single
author_profile: true
read_time: true
comments: null
share: false
categories:
  - DevOps
tags:
  - iot
  - hassio
  - docker
  - hosting
---

A lot of time passed since I created these configs, but I hope I can summarize them here!

I have a small home-server. It was build about 3 years ago (<300$, <40W). The specs are:
 - [cheapest small PC case](https://www.lc-power.com/en/product/pc-cases/mini-itx-cases/lc-1400mi/) I could found locally
 - Integrated CPU on an Asrock motherboard ([probably this](https://asrock.com/mb/Intel/J3160-ITX/index.us.asp))
 - 8Gb ram
 - 16Gb usb3 pendrive (as a system partition)
 - 2Tb hdd (mounted as `/hdd`)
 - I changed the PSU after 2 years (mostly bcs of the noise)
 
I needed to buy a [usable router](https://www.asus.com/Networking/RT-AC66U-B1/) too in the last year or so, 
because the one I get from my internet provider couldn't 
handle 6-8 devices without a crash...

I installed Ubuntu 16.04 server version. Configured the /hdd mount point.
Checked the networking settings, the ssh config, and placed it to the top of the lobby-cabinet.

Installed [docker](https://docs.docker.com/install/linux/docker-ce/ubuntu/), and [docker-compose](https://docs.docker.com/compose/install/).

I am using two docker-compose files. One for the "file-server" things, and one for the "home-automation" things.

The file-server is the easy one:
{% gist c2817c767fdf8f348157797615825baa file-server.compose.yml %}
I use deluge to get legal content from torrents (like linux distros), and miniDLNA to index them and send to my tv over wifi.
They both getting restarted after a server restart, and they both need the `network_mode: host` for the proper result.

MiniDLNA will provide a http interface on port 8200, and the Deluge UI came up on 8112.

The hassio one has a bit more component:
{% gist c2817c767fdf8f348157797615825baa hassio.compose.yml %}
Beside the `home-assistant`, I have an `mqtt` and a `tasmoadmin` container.
I configured the mqtt to be priviledged and restarted, and leave the other 
two as close to the original configuration recommendation as I could.
Configuring the hame-assistant is not an easy task and you want to follow the original documentation with it.

I use make-files most of my projects so I have one here too.
{% gist c2817c767fdf8f348157797615825baa Makefile %}
Mostly for easier command execution, I can start/update the filestack with `make fup`.

I will need a monitoring system... 
I had munin before, but I would rather go with a node-exporter, prometheus, grafana trio.
The problem with the monitoring hosted on the same server as we want to monitor is pretty obvious...
And it is resource heavy if we consider that I didn't checked the server in the last ~100 days.

Another interesting idea was to make a gitrepo from my docker folder, and push the modifications time-to-time.
I'm not doing this right now, but probably worth a try if you want to reproduce your stack fast and easy.
(I'm doing this on some prod environments for version traceability.)

I'm pretty happy with this setup so far. It's up and running in the last 106 days without any issue,
and it was born under 3h from a complete reinstall.
My home-assistant tracking sensor data, my deluge contributing back to the open-source community, 
and my tv can play back the data from my winchester.

