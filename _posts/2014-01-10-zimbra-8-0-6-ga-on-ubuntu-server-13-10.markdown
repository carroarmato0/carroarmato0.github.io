---
layout: post
title: Zimbra 8.0.6 GA on Ubuntu Server 13.10
date: '2014-01-10 23:25:21'
tags:
- saucy
- ubuntu
- zimbra
- ubuntu-13-10
---

The current version of Zimbra at the time of writing this is does not support Ubuntu 13.10, but rather Ubuntu 12.04 LTS. Yes, there is a reason for chosing an LTS release, but if you ended up on this blog post, it's because you like living on the edge and not doing this in a corporate environment for production purposes. Riiiggghht? Just checking.

In order to install the latest and greatest of Zimbra on Saucy, you need two things:

- **libgmp3c2** which isn't present for Saucy, but the one from Quantal is just fine. http://packages.ubuntu.com/quantal/amd64/libgmp3c2/download

- **libencode-perl**

For the lazy bums out there:

	# wget http://mirrors.kernel.org/ubuntu/pool/universe/g/gmp4/libgmp3c2_4.3.2+dfsg-2ubuntu1_amd64.deb
    # sudo dpkg -i libgmp3c2_4.3.2+dfsg-2ubuntu1_amd64.deb
    # sudo apt-get install libencode-perl
    
   And proceed with the installation of zimbra with **./install.sh --platform-override**
   
   
## Fix monitoring
Zimbra monitors itself by looking for entries in **/var/log/zimbra.log** and **/var/log/zimbra-stats.log**
It does that by adding entries in **/etc/rsyslog.conf**

	...
    #
	# Include all config files in /etc/rsyslog.d/
	#
	$IncludeConfig /etc/rsyslog.d/*.conf
    ...
	local0.*                -/var/log/zimbra.log
	local1.*                -/var/log/zimbra-stats.log
	auth.*                  -/var/log/zimbra.log
	mail.*                  -/var/log/zimbra.log

However it looks like rsyslog is not able to write to that. I temportary fixed that by changing the group ownership to **syslog**, but this only seems to work right until the logs are being rotated and the new file has the original zimbra group.

	

Happy mailing!