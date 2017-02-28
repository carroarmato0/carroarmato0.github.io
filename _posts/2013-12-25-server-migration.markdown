---
layout: post
title: Server Migration
date: '2013-12-25 13:49:26'
tags:
- lxc
- puppet
- linux
- migration
---

If you are reading this, it means that you've reached my new dedicated server and the DNS updates have reached a DNS server near you ;)

This post focuses on what I've done and the challenges I encountered along the way.

Puppetized
---

Since I've learned how to use [Puppet](http://puppetlabs.com/puppet/puppet-open-source) at work, I see the benefit of using it on my server. I've put a lot of effort configuring everything I want, choosing the best modules which fit my needs and added the glue to make them all work together where needed.

####Why? 

* What if I need to migrate to another server tomorrow and redo all the work from scratch and forget some conf files?

* What were those shady options again?

* Oh god I don't want to spend the whole day dealing with x and y again.

The goal is to be able to redeploy much faster the next time and have consistency.

####How?
I use [GIT](http://git-scm.com/) to manage all the manifests and submodules for the modules themselves which I host through [GitLab](http://gitlab.org/).
I've installed [GitLab's Continuous Integration](http://gitlab.org/gitlab-ci/) Server hoping to have tests run on my code before pushing it to the Puppet Master.

Using [Foreman](http://theforeman.org/) for getting reports back of every Puppet run.

Instead of running virtual machines, I've opted to heavily use [LXC](http://linuxcontainers.org/) which is a container technology allowing me to start machines up in a matter of seconds with their own LVM LV and with a lower footprint since I don't have the overhead of virtualization.
I couldn't find any decent LXC puppet module at the time, so I opted to write my own which can be found at [https://github.com/carroarmato0/puppet-lxc](https://github.com/carroarmato0/puppet-lxc).
This means I can manage LXC containers straight from Puppet, and since I made some slight modifications to the cached containers, the containers are already Puppet ready and report back to the Puppet Master ready to be deployed.

On the physical machine I use [Pound](http://www.apsis.ch/pound) for Reverse Proxying vhost requests to the proper container running the appropriate service.
Again I wasn't too happy with the existing Puppet modules for Pound, so I opted to write my own: [https://github.com/carroarmato0/puppet-pound](https://github.com/carroarmato0/puppet-pound).
Although it's only been tested on my server (Ubuntu), I'm pretty sure it's easy to add support for other systems. One of the main advantages of my module is the ease of use and can easily be extended with the rest of the options Pound has.

####Challenges:

#####Package versions vs Puppet modules

One of the first things I noticed while Puppetizing my Ubuntu 13.10 server, **but this is true for any other new iteration of a distro**, is that **newer package versions** sometimes aren't recieved well by existing Puppet modules.

Some examples:  

* **Apache 2.4**: the existing puppet modules are still in the process of planning the necessary changes.

* **OpenVPN 2.3.2**: OpenVPN decided to no longer ship with easy-rsa which the [most popular OpenVPN puppet module](https://github.com/luxflux/puppet-openvpn) (I could find) relies on.

* ...

**Solutions?**

* I could have sticked with a **Long Term Support** edition.

* I could rewrite the modules with the necessary changes and send pull requests.

* ~~I could write my own Puppet module which only scratches my itch.~~

Point 2, while being theoretically the best option, means you're dealing with humans with their own opinions, and this might slow down the whole process of adding the patch. In the best case, the patch is added quickly for the whole world to enjoy. Worst case: everyone forks and continue on their own.

#####Apply glue to different Puppet modules

One of the issues I've noticed as I've been window shopping Github for various Puppet modules, is that often there are too many different modules which try to accomplish the same thing with various degrees of success and quality.

The temptation of writting your own thing to scratch your own itch is very high rather than contribute patches back to already existing ones.

I had this with the [puppet-nginx module](https://github.com/jfryman/puppet-nginx) of James Fryman, I restrained myself from doing any modifications and use what options where available to get php5-fpm working with the vhosts. But I needed to migrate a **Gallery3** website which needed the following to have it work (among things):

	location / {
    	root  /var/www/somegallery3site.carroarmato0.be;
        index  index.html index.htm index.php;
        
        if (-f $request_filename) {
        	expires max;
            break;
        }
        if (!-e $request_filename) {
        	rewrite ^/(.+)$ /index.php?kohana_uri=$1 last;
        }
    }

Turns out that the module doesn't allow me to add arbitrary chunks of code inside a ```location``` section without having it add ```;``` at the end.
Nginx obviously chokes if the above code ends with ```};```

I looked between the issues registered with the module and it seems like that support is coming, but I needed those changes **now**.
So essentially what I did was **implicitly fork** the project in the hope that support will be added upstream. I could have formally forked it and send in a pull request, but I know it's not going to happen since they are already working on something, if not, already have but need to tag the release.

**Everytime I fork something, I die inside.** Because essentially I'm taking something which is referenced and used by many people with (mostly) the intent of adding my own features and hopefully have my pull requests accepted, and splintering the effort. If that's not the case or upstream isn't able to provide a better alternative, why would I abandon my fork?

This is tiny rant on how it seems that the whole process actually promotes splintering efforts. A company using Puppet will have its own modules and its own way of arranging the code, which is very likely different at another company.


Other than all that, I'm pretty satisfied with the outcome. Although at first it might seem like using Puppet is overkill for just one server, I'm already enjoying the fruits of my endeavour, committing tiny changes in the manifests which will take care of repetitive changes with little effort, not to mention starting up LXC containers which get basic configurations applied immediately like adding my user, SSH key and security settings.