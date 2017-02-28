---
layout: post
title: Updating CRL with Easy-RSA
date: '2014-08-29 09:03:17'
tags:
- ssl
- crl
- easy-rsa
- client-side-authentication
- apache
---

At work I've setup **Client Side Certificate Authentication** to protect a sensitive website for HR since the built-in authentication mechanism left more to be desired.

I'm going to skip the part about how I've set it up, but the important part is that I used **easy-rsa** to make the management of the PKI a lot easier and that Apache is configured to check the **certificate revocation list**.

All of a sudden I got the message that the certificate authentication mechanism suddenly blocked everyone from accessing the website.

After taking a look at the Apache logs I noticed that all certificates had been revoked due to the **CRL being out of date**:

    [Fri Aug 29 10:04:25 2014] [error] [client 84.xxx.xxx.xxx] Certificate Verification: Error (12): CRL has expired
    [Fri Aug 29 10:17:45 2014] [warn] Found CRL is expired - revoking all certificates until you get updated CRL

I used the following command to inspect the **crl.pem** file:

    openssl crl -in keys/crl.pem -text
    Certificate Revocation List (CRL):
        Version 1 (0x0)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: /C=xx/ST=xx/L=xx/O=xx/OU=xx/CN=xx.xx.be/name=xx_CA/emailAddress=support@somethingsomething.eu
        Last Update: Jul 29 14:12:10 2014 GMT
        Next Update: Aug 28 14:12:10 2014 GMT

As you can see it had expired (Next Update)

I tried looking on the net to find a procedure but couldn't find anything useful other than maybe the proper openssl way:

    openssl ca -gencrl -out keys/crl.pem -config openssl-1.0.0.cnf 

Unfortunatly this gave me the following error:

    Using configuration from /usr/share/ca-certificates/knet/openssl-1.0.0.cnf
    error on line 145 of config file '/usr/share/ca-certificates/xxx/openssl-1.0.0.cnf'
    139760597432136:error:0E065068:configuration file routines:STR_COPY:variable has no value:conf_def.c:629:line 145

What a bummer....
So next I tried figuring out how I could trick the whole setup into updating that file.

I remember I had a **dummy certificate** laying around for testing purposes.

    ./revoke-full dummy
    Using configuration from /usr/share/ca-certificates/xxx/openssl-1.0.0.cnf
    ERROR:Already revoked, serial number 02
    Using configuration from /usr/share/ca-certificates/xxx/openssl-1.0.0.cnf
    dummy.crt: C = xx, ST = xx, L = xx, O = xx, OU = xx, CN = dummy, name = xx, emailAddress = xx@somethingsomething.eu
    error 20 at 0 depth lookup:unable to get local issuer certificate

Although this seems like it did nothing because I already had revoked it in the past, it did however update the CRL file!

    ./list-crl 
    Certificate Revocation List (CRL):
        Version 1 (0x0)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: /C=xx/ST=xx/L=xx/O=xx/OU=xx/CN=xx.xx.be/name=xx_CA/emailAddress=support@somethingsomething.eu
        Last Update: Aug 29 08:32:39 2014 GMT
        Next Update: Sep 28 08:32:39 2014 GMT

Hurray, it's valid for another month.

Next I reloaded Apache and everything was working again.

I added a **cronjob** which would re-revoke the dummy certificate as a workaround to keep the crl fresh.

Hope this was useful to others using a similar setup.