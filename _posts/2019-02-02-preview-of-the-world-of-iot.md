---
title: Preview of the "World of IoT"
layout: single
author_profile: true
read_time: true
comments: null
teaser: ""
share: true
categories:
  - IoT
tags:
  - iot
  - intro
---

Every year when I get the two weeks of Christmas holiday I start to play with/research/learn something new. Last year it was docker, and the basics of system operation (metrics, log aggregation, etc.). [(If you are interested you can check it out here!)](https://leaks.wanari.com/2018/02/12/rancher-grafana-devops-case-study/) When I moved into my appartment I already knew that this year will be about the smart devices!

So I started with basic internet research. What options we have if we really want to make our home smartish? We have a lot of options. And a lot of really bad options... I found [@internetofshit](https://twitter.com/internetofshit) on twitter, and I get really mixed feelings about the whole "premade, you just need to buy it and download the app" concept. If I buy a smart-outlet from providerA and a motion sensor from providerB there are absolutely no "easy way" to connect them. So either you hack them (I will touch this topic later), you try to live with them (with 4-6 apps, maybe some hacked up IFTT which only works when your phone works), or buy all the stuff from one provider.

At 2011 I was in a project where we thinked about IoT (this word was not even existed back then), and tried to make "smart" sensors for a closed environment. It is even more interesting to me, how much the technology and the availability of the technology moved till then. But our biggest problem is still a huge problem in this new industry.

# Lets talk about security!

When you let a new smart device to connect to your network, it is somewhat like when you give keys to a stranger to your house.  The "smart" means, that you can control the thing if you have internet access. But for a non technical person its not obvious that you don't directly communicates with the device. You and your device data meets somewhere on the internet, because most of the internet providers hide your home network from the outside world (and even when they not doing this you need to know how to configure your router to make the device visible). So for the device manufacturers, the easiest way to host a server. Every device send data to that server, and every user with the mobil app speaks with that server too. So you not only need to trust that the device will not harmful on its own, but you need to trust that the manufacturer can handle and secure the data correctly.

With a humidity+temperature sensor the harmful manufacturer can scan your network, see your network drives, or just "listen" the network. But with a harmful attacker and a "not so good" cloud system can offer the attacker methods to switch your lamp, see what your camera sees, or let himself in with a smart-lock, even if the manufacturer is "trusted".

For example [this post](https://shkspr.mobi/blog/2016/03/the-absolute-horror-of-wifi-light-switches/) is about a cheap chinese light-switch which can be switched by anyone who was smart enought to check how the communication works. But Samsung had problems with their [bulbs](https://www.wired.co.uk/article/crypto-weakness-lightbulbs) and [hubs](https://threatpost.com/bugs-in-samsung-iot-hub-leave-smart-home-open-to-attack/134454/) too.

How we can minimalize these attack-vectors? Good question! After reading a bunch of horror-stories, I made two decisions. Firstly, I will not let any "cloud-based" device to my network. And secondly, I will prefer wifi over everything. (BT and ZigBee are not seems as reliable for me right now.) This means, I can't use about the 90% of the prebuild/commercial IoT devices.

# There is hope
After this security consideration, I started to modify my research directions. 

There are several "pro-teams" whoes made all the things for you for good money. They are most of the time came from the DIY scene and make the hobby to a living. They use selfbuild devices, and make your system up-and-running. Teach you how to use it, and fix it for you if there is an error later on. They are not easy to find, and really priciy. But at least you don't need to know anything about the IT world. And they can tell you why they are doing what. (So it's not a random reseller in the shop who learned that the blue laptop is better because it has more ram, but has no idea what ram means.)

There are several DIY, and hack yourself forums, where you can learn how you can build (or modify a commertial) smart device. How to make them communicate with each other. How to make them secure. In theory this is the cheapest way to build a smart-home. In reality you need to learn a lot if you are not familiar with the IT world.

# DIY it is...
I choosed the DIY way, because it seems to be fun!

I need to learn (or learn more about):
* soldering
* basic electronics (I know it in theory but its not the same as working with the actual wire)
* microcontroller programing (I did this too before, but deffinitly not a pro)
* mqtt
* openVPN
* maybe some google-home hacking

My stack probably will be:
* Sonoff T1 light-switches (commertial hw)
* [Sonoff-tasmota](https://github.com/arendst/Sonoff-Tasmota) rom
* Wemos D1 mini (developer board/hw)
* Sensors supported by the tasmota rom (see at the next post)
* A new router with VPN support
* My (right now) file server (as a server)
* One of my cloud servers (as vpn bridge)
* Docker
* Mosquitto MQTT Broker
* [HASSIO](https://www.home-assistant.io/) (as a UI/management)
* In the future I want a screen too (not decided yet, maybe a cheap tablet)

I already orderd and recieved the D1 mini, and some sensors (next post). I have the router, and ofc. the two servers. With the choose of the prebuild rom this will more like to configuring/installing the things than software development.
