---
title: OpenVPN, Asus router, VPS - how to reach your home network
layout: single
author_profile: true
read_time: true
comments: null
share: false
categories:
  - DevOps
tags:
  - vpn
  - docker
  - hosting
---

Yet another OpenVPN tutorial, which would save me 3 hours.

## Motivation
I have some servers in the cloud. They have public ip, and I have dns to them.
I have a private LAN at home. There are and will be several devices in this network, like sensors, smart plugs, and I have a homeserver too.

I wanted a solution to reach my home things from anywhere without thretening them. 

As I progressed I found that what I want is; a solution which can easily add my laptop (and probably my phone) to my home network from any network.  

## Options

The [simple dyndns](https://www.howtogeek.com/66438/how-to-easily-access-your-home-network-from-anywhere-with-ddns/) way to connect to your home network is messy, and you need portforwarding, and I hate the whole process.

The [ssh](https://medium.com/botfuel/how-to-expose-a-local-development-server-to-the-internet-c31532d741cc) way is messy, you need to config a lot of things manually and not really reliable.

The OpenVPN with dyndns a bit better, but for "customer safety" my NAT is behind at least one other NAT (so this will not work for sure, I love the internet-providers :D ).

Use a remote server as VPN server, and use my router as a client which exposes its whole NAT to other VPN clients.
(Ofc. if your router has no vpn functionality build in, you can use an inner server too, but you need probably more forwarding options.)

## Solution

I found a good docker image: https://github.com/kylemanna/docker-openvpn

Some [posts](https://community.openvpn.net/openvpn/wiki/RoutedLans#no1), and [discussions](https://serverfault.com/a/767186),  and [error-logs](https://www.sparklabs.com/support/kb/article/bad-compression-stub-decompression-header-byte/) and other [discussions](https://www.reddit.com/r/qnap/comments/6zjbbb/need_to_setup_openvpn_as_splittunnel_do_not_want/) and multiple iterations later I get it working with the following commands:
(My home network is in the 10.32.45.0/24 range.)

```
mkdir vpn
cd vpn
docker run -v $PWD:/etc/openvpn --log-driver=none --rm kylemanna/openvpn ovpn_genconfig -u udp://VPN.SERVERNAME.COM
docker run -v $PWD:/etc/openvpn --log-driver=none --rm -it kylemanna/openvpn ovpn_initpki
docker run -v $PWD:/etc/openvpn --log-driver=none --rm -it kylemanna/openvpn easyrsa build-client-full router nopass
docker run -v $PWD:/etc/openvpn --log-driver=none --rm -it kylemanna/openvpn easyrsa build-client-full laptop nopass
docker run -v $PWD:/etc/openvpn --log-driver=none --rm kylemanna/openvpn ovpn_getclient router > router.ovpn
docker run -v $PWD:/etc/openvpn --log-driver=none --rm kylemanna/openvpn ovpn_getclient laptop > laptop.ovpn
```
Starting with the commands from github! They need to be executed separately, and most of them needs some interactive user config.


```
sed -i '' '/dns/s/^/#/' openvpn.conf
sed -i '' '/dhcp/s/^/#/' openvpn.conf
```
I commented out all the dns and dhcp configs, because I don't need them. (If you have an inner name server, you probably want these dns settings in the client config files rather than the vpn-server in this use-case.)


```
echo route 10.32.45.0 255.255.255.0 >> openvpn.conf
echo push \"comp-lzo no\" >> openvpn.conf
echo push \"route 10.32.45.0 255.255.255.0\" >> openvpn.conf
echo client-to-client >> openvpn.conf
echo client-config-dir ccd >> openvpn.conf
echo iroute 10.32.45.0 255.255.255.0 >> ccd/router
```
The next block is setting up the client-to-client communication and iroute. For more information and better understanding please read through [this](https://community.openvpn.net/openvpn/wiki/RoutedLans#no1).

```
sed -i '' '/redirect-gateway/s/^/#/' *.ovpn
echo comp-lzo no >> router.ovpn
echo comp-lzo no >> laptop.ovpn
echo keepalive 10 60 >> router.ovpn
echo keepalive 10 60 >> laptop.ovpn
```
I don't need the redirect-gateway, because I only want to redirect the traffic to my LAN ip-range, others should leave the VPN alone.
Added back some keepalive and comp-lzo options for less warnings and better stability.

```
sed -i '' '/nobind/s/^/#/' laptop.ovpn
```
I have no idea why I needed to comment out the nobind, I read multiple things and still don't understand what it does.

```
scp -r ./* clappy:/nfs/lan-openvpn-pvc-PVCID
```
I moved my locally created files to my servers persistent volume, after I started my container.
(clappy is the name of my server configured in the ~/.ssh/config)

After a container restart, and happy log messages, you need to upload your router config to your router (or homeserver). 
(I have an asus router, it has a not so complicated interface, vpn -> client -> addprofile -> browse (and upload) -> finish style of process)

If the router can connect to your vpn server as a client, you load your laptop config into tunnelblick, and you are ready to go!




