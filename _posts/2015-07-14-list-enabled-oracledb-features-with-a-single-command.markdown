---
layout: post
title: List and test enabled OracleDB features using the commandline
date: '2015-07-14 13:24:21'
---

While automating the deployment and configuration of Oracle 12c installations, I've had a need to be able to detect and automatically disable certain features which require extra licensing fees.

Some of these extra features graciously enabled by default by Oracle are:

- Real Application Testing (rat)
- Partitioning (partitioning)
- On-line Analytical Processing (olap)
- Data Mining (dm)

One way of checking if these options are enabled is by connecting to the database and see if those are listed in the banner:

> oracle@hostname:~> sqlplus / as sysdba 
> 
> SQL*Plus: Release 12.1.0.2.0 Production on Tue Jul 14  0:47:15 2015 
> Copyright (c) 1982, 2014, Oracle. All rights reserved.

> Connected to: 

> Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production 

> With the ==Partitioning==, Automatic Storage Management, ==OLAP==, ==Advanced Analytics== and ==Real Application Testing options==

Unfortunately it's not easy or reliable to parse that output.

One way which I've found to get a proper listing was to use ***sqlplus*** (Note: in the examples it's implied that you have an ORACLE_SID set containing the name of the database needed).

    oracle@hosname:~> sqlplus -L / as sysdba <<< 'select * from v$option;'

    ...
    SQL> 
    PARAMETER
    ----------------------------------------------------------------
    VALUE								     CON_ID
    --------------------------------------------------------------------------
    Partitioning
    FALSE									  0

    Objects
    TRUE									  0

    Real Application Clusters
    FALSE									  0


    PARAMETER
    ----------------------------------------------------------------
    VALUE								     CON_ID
    --------------------------------------------------------------------------
    Advanced replication
    TRUE									  0

    Bit-mapped indexes
    TRUE									  0

    Connection multiplexing
    TRUE									  0

    ...

This is a bit better, though it needs some work to properly parse.

    oracle@hostname:~> sqlplus -L / as sysdba <<< 'select * from v$option;' | grep -A 1 '^Real Application Testing$' | grep TRUE

    oracle@hostname:~> sqlplus -L / as sysdba <<< 'select * from v$option;' | grep -A 1 '^Partitioning$' | grep TRUE

    oracle@hostname:~> sqlplus -L / as sysdba <<< 'select * from v$option;' | grep -A 1 '^OLAP$' | grep TRUE

    oracle@hostname:~> sqlplus -L / as sysdba <<< 'select * from v$option;' | grep -A 1 '^Data Mining$' | grep TRUE

These commands can be used to provide the correct return code to indicate if any of the above listed features are enabled in a script.

To disable these features, you will need to shutdown every database running in the current oracle home and afterwards restart it.
This can also be done with the following commands:

    oracle@hostname:~> sqlplus -L / as sysdba <<< 'shutdown immediate'

    oracle@hostname:~> chopt disable rat
    oracle@hostname:~> chopt disable olap
    oracle@hostname:~> chopt disable partitioning
    oracle@hostname:~> chopt disable dm

    oracle@hostname:~> sqlplus -L / as sysdba <<< 'startup'
