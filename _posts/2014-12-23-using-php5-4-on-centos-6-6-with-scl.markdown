---
layout: post
title: Using PHP5.4 on Centos 6.6 with SCL
date: '2014-12-23 09:57:27'
---

The Software Collections ( **SCL** ) Repository is a blessing for people wanting to use more recent versions of popular system software like Python, PHP or Ruby.

I've given it a try by updating a virtual machine running an **Owncloud** instance with **PHP 5.3.3**.

First up I enabled the SCL repository (for your convenience I left out the part related to using your own repository management system).

	yum install centos-release-SCL
    yum clean all
    yum makecache
    
Then I've lookup the already installed php software to figure out what php 5.4 counterparts had to be installed.

	# rpm -qa | grep php
    php-common-5.3.3-40.el6_6.x86_64
    php-pdo-5.3.3-40.el6_6.x86_64
    php-pear-MDB2-2.5.0-0.9.b5.el6.noarch
    php-pear-Net-Curl-1.2.5-4.el6.noarch
    php-bcmath-5.3.3-40.el6_6.x86_64
    php-cli-5.3.3-40.el6_6.x86_64
    php-xmlrpc-5.3.3-40.el6_6.x86_64
    php-pear-1.9.4-4.el6.noarch
    php-mysql-5.3.3-40.el6_6.x86_64
    php-process-5.3.3-40.el6_6.x86_64
    php-pear-MDB2-Driver-mysqli-1.5.0-0.8.b4.el6.noarch
    php-xml-5.3.3-40.el6_6.x86_64
    php-gd-5.3.3-40.el6_6.x86_64
    php-mbstring-5.3.3-40.el6_6.x86_64
    php-5.3.3-40.el6_6.x86_64
    php-devel-5.3.3-40.el6_6.x86_64
    php-ldap-5.3.3-40.el6_6.x86_64
    
 And then installing the PHP 5.4 packages:
 
	yum install php54-php-bcmath php54 php54-php-cli php54-php-gd php54-php-ldap php54-php-mbstring php54-php-xmlrpc php54-php php54-php-pdo php54-php-pear php54-php-pecl-apc php54-php-mysqlnd php54-php-process
    
Then I had to **enable** the system to use the new PHP version by default:

	# source /opt/rh/php54/enable
	# php -v
	PHP 5.4.16 (cli) (built: Nov 19 2014 08:05:17) 
	Copyright (c) 1997-2013 The PHP Group
	Zend Engine v2.4.0, Copyright (c) 1998-2013 Zend Technologies

**Apache** required an extra step to make it use the newer version.

You can either remove the appropriate package, or, simply remove the old php.conf file.

	# rm -fr /etc/httpd/conf.d/php.conf
    # service httpd restart
    Stopping httpd:                                                        [  OK  ]
    Starting httpd:                                                        [  OK  ]
