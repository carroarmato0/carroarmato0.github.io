---
layout: post
title: Setting up a Lan Party Gateway - Part 1
date: '2014-04-11 16:12:36'
tags:
- gateway
- iptables
- mptcp
- lan-party
- openvpn
- squid
- linux
- proxpn
- dnsmasq
---

This will be a series of blog posts on my journey into setting up a Lan Party Gateway.

I've set myself the following goals on what the device should achieve and be capable of:

- **DNS**
- **DHCP**
- **VPN loadbalancing**
- **Transparent cache Proxy**

For future developments I'd like to add Capture Portal functionality, not just to redirect everyone's connection for the first time to see a disclaimer, but also to be able to get a dropdown list with a selection of VPNs so that the user can decide which one he/she would like to use to bypass certain country restrictions.

Here's a list of which technologies I'm using to achieve the above mentioned goals:

- DNS + DHCP: **dnsmasq**
- VPN: **OpenVPN**
- Transparent cache Proxy: **Squid**

The above will be running on a **Shuttle XPC XH61V**, with an Intel Core **I3**-3240, Crucial SoDIMM DDR3 **8 GB** 1333 MHz and a Kingston **60 GB SSD** 2.5" SATA3 V300 drive.
The important part is having **2 Gigabit** network **interfaces**, 1 for the wan, the other for the local lan.

![Shuttle Front](/content/images/2014/Apr/IMG_20140411_181317.jpg)
![Shuttle Back](/content/images/2014/Apr/IMG_20140411_181418.jpg)

Since I need to provide clients connecting to the internal lan with an IP address (and the rest of the relevant networking information), and like to have them be able to use a **local DNS domain** to help them identify themselves, I've opted for dnsmasq which is very simple to use, but also handles both DHCP and DNS in one easy to use setup.

I am a **ProXPN** customer, which means I have at least 12 VPNs at my disposal (6 OpenVPN and 6 PPTP links) with endpoints in different countries (and not mentioning that those VPNs also listen of different ports, so arguably that's more than 12).

Since I'm very well accustomed to OpenVPN, and because it's a better technology in general I'll be using that rather than PPTP.

Had to rely on different configuration sources for setting it up with ProXPN, eventhough they don't officially support Linux, after some digging through the internet it's very possible to connect through them.

Since people have a selfish tendency to use Lan Parties as an excuse to start **downloading massive amounts of data** on the Internet causing **bandwidth strain**, **Squid** will have a **dual function**. Not only will it be **caching** the **http** content, but it will also be **filtering traffic**, **blocking** certain **unwanted sites** and in combination of some firewall policies, get a grip on torrent clients that happen to be running.

In order to try to be a good internet citizen, eventhough I have an unlimited traffic through my ProXPN subscription, I feel like I need to take this a step further, rather than sending and receiving all connections through 1 single VPN, I wanted to have all my **tunnels** available in a **loadbalanced** fashion.

## DNS/DHCP - dnsmasq


Setting up dnsmasq is very easy, I used the following configuration:

    # Prevent non-routable private addresses from being forwarded
    bogus-priv
    # Set domain
    domain=lan
    # Set listening interface, dhcp range and lease time
    dhcp-range=eth1,192.168.200.5,192.168.200.254,24h
    # Listen on specific interface
    interface=eth1
    # Prepend domain to all hosts
    expand-hosts
    # Queries for private domain only answered by local dns
    local=/lan/
    # Pass default gateway
    dhcp-option=eth1,3,192.168.200.1
    # Pass DNS server
    dhcp-option=eth1,6,192.168.200.1
    # Pass search domain
    dhcp-option=eth1,119,lan
    # Pass NTP server
    dhcp-option=eth1,42,be.pool.ntp.org
    # Upstream DNS servers
    server=8.8.8.8
    server=8.8.4.4
    # Only bind to interfaces that it's listening on
    bind-interfaces
    # Be authoritative and barge in when a machine wakes up and broadcasts's a dhcp request
    dhcp-authoritative
    # For debugging purposes, log each DNS query as it passes through dnsmasq.
    #log-queries
    # Log lots of extra information about DHCP transactions.
    #log-dhcp
    # Log location
    log-facility=/var/log/dnsmasq.log

After connecting a client to the eth1 directly, lo and behold it got all the required information.

As a temporary means for the clients to be able to connect to the internet through my gateway, I had to configure the following in iptables and to allow the traffic coming from the lan interface to be forwarded:

    sysctl net.ipv4.ip_forward=1

    iptables -t nat -j MASQUERADE -o eth0 -s 192.168.200.0/24

Next step, having that traffic go through the VPN.


## VPNs - OpenVPN


While setting up just 1 VPN is very straight forward, I've found out that setting up multiple ones (10) with the intention of loadbalancing through them proved to be more complicated than I originally thought.

**My original thought** was: 'Every VPN has its own virtual interface, I could just **pass all traffic** coming **from the lan through them**.'

While my idea was grand, the reality is a bit more complicated than that.

I was imagining things more trivially, forgetting for a moment about how routing works in Linux, and basic tcp.

After cooling down and giving this more thought, I figured that I had to **add extra routing** information for **each VPN link**. For the **loadbalancing** part, I figured that I could use **Multipath TCP** for this.

Since I haven't come around to that, I'll list the different ways I've tried to set things up and the various issues that I've encountered.


### Bridging the interfaces

Wishful thinking...

### One configuration file per link

After starting OpenVPN, as the links come up succesfully, the tunnel devices start appearing:

	[carroarmato0:~] $ ip link
    6: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 100
    link/[65534]
	7: tun1: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 100
    link/[65534]
	8: tun2: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 100
    link/[65534]
	9: tun3: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 100
    link/[65534]
	10: tun4: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 100
    link/[65534]
	...
    
I think this is where I should continue to focus on.

Here are some issues that I've encountered in this setup:

#### Accepting routes from all VPN servers

This is bad, really bad. Every server is going to modify the routing on the system, pointing to itself as the default gateway.

In a way I've reached tunnelception. This only works for 2 tunnels and then all the other tunnels aren't able to start because of routing mayhem.

I was able to fix this issue by adding this entry in all conf files:

	# Block servers adding their routing information on the client
	route-nopull
    
This was an indicator that, if I wanted to go through with configuring them in a loadbalanced fashion. That I would have to somehow add the routing info myself or through scripting. Adding the default gateways settings myself per tunnel interface, but not globally.

### All remotes defined in one configuration file

In the documentation of OpenVPN, they have a section about [implementing a load-balancing/failover configuration](https://openvpn.net/index.php/open-source/documentation/howto.html#loadbalance).

This means defining all your endpoints in one configuration file like so: 

	remote 1.1.1.2 8080
    remote 2.2.2.2 443
    remote 5.5.4.2 80
    ...
    
This results in only one tunnel interface being created. OpenVPN depending on the configuration will either try the different remotes in order or randomly until it is able to establish a succesful connection.

I haven't been able to check if this is really loadbalancing anything, but one thing for sure is that this provides failover in case the active tunnel goes down.

This implies accepting the route changes pushed by the remote server by uncommenting or removing this configuration line:

	# Block servers adding their routing information on the client
	# route-nopull
    
All traffic will then be forwarded through the tunnel.

	[root@zuul openvpn]# ip route
	173.0.14.249 via 10.0.42.1 dev eth0   <--- real gateway on wan
	173.0.10.0/24 dev tap0  proto kernel  scope link  src 173.0.10.72 
	10.0.42.0/24 dev eth0  proto kernel  scope link  src 10.0.42.173 
	169.254.0.0/16 dev eth0  scope link  metric 1002 
	169.254.0.0/16 dev eth1  scope link  metric 1003 
	default via 173.0.10.1 dev tap0       <--- vpn's gateway


That's it for this first blog post.
Will try advancing from here, so expect to see a follow up on this one :)