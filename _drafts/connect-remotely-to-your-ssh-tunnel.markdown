---
layout: post
title: Connect remotely to your ssh tunnel
---

I had a colleague today trying to tunnel through SSH on a local LXC container and wanting to access the remote LDAP server through it.

A quick recap of what SSH tunnelling is: it allows you to access or provide a network service that the underlying network does not support or provide directly. 

Most people acquainted with SSH will already know how to use this and connect to the remote service by connecting to a specific port now opened by the tunnel locally to reach the remote service.

But what if you wanted to access that tunnel remotely?

Turns out **by default**, **SSH binds** the tunnel port **to localhost**.

*Example*:

```
ssh -f -L 9000:ldap.example.com:389 jumphost.example.com -N

-f:     force the connection to the background
-L:     specify the connection to be made between your machine and the remote host (jumphost)
9000:   the port which will be your end of the tunnel
ldap:   the hostname of the actual machine running the service to which you want to connect to
389:    the port to the remote service you want to connect to, the other end of the tunnel
jumphost: the intermediary machine which you can connect to directly
-N:     don't execute a remote command, we just want the port to be forwarded
```

By using the **-g** option, you can allow remote hosts to connect to local forwarded ports by specifying a **bind address**, usually **0.0.0.0** if you want to be able to connect to it from any interface.

```
ssh -f -gL 0.0.0.0:9000:ldap.example.com:389 jumphost.example.com -N
```