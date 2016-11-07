---
# required metadata

title: Troubleshoot SQL Server on Linux | SQL Server vNext CTP1
description: 
author: rothja 
ms.author: jroth 
manager: jhubbard
ms.date: 11-03-2016
ms.topic: article
ms.prod: sql-non-specified
ms.service: 
ms.technology: 
ms.assetid: 

# optional metadata

# keywords: ""
# ROBOTS: ""
# audience: ""
# ms.devlang: ""
# ms.reviewer: ""
# ms.suite: ""
# ms.tgt_pltfrm: ""
# ms.custom: ""
---
# Troubleshoot SQL Server on Linux

## Introduction

This document describes how to troubleshoot Microsoft SQL Server running on Linux or in a Docker container. When troubleshooting SQL Server on Linux, please make remember the limitations of this private preview release. You can find a list of these in the [Release Notes](sql-server-linux-release-notes.md).

## Support

update this to reflect support for public preview

## Managing the SQL Server Service (start, stop, restart, status)

### Managing the mssql-server service in Red Hat Enterprise Linux (RHEL) and Ubuntu 

Check the status of the status of the SQL Server service using this command:

    $ systemctl status mssql-server

You can stop, start, or restart the SQL Server service as needed using the following commands:

    $ sudo systemctl stop mssql-server

    $ sudo systemctl start mssql-server
    
    $ sudo systemctl restart mssql-server

### Managing the execution of the mssql Docker container

You can get the status and container ID of the latest created SQL Server Docker container by running the following command (The ID will be under the “CONTAINER ID” column):

    $ docker ps -l

You can stop or restart the SQL Server service as needed using the following commands:

    $ docker stop <container ID> 

    $ docker restart <container ID> 

You can run a new container by using the following command:

	$ docker run –e "ACCEPT_EULA=Y" –e "SA_PASSWORD=<your password>" [OPTIONS] private-repo.microsoft.com/mssql-private-preview/mssql-server 

    > [!IMPORTANT]
    > The following steps will change when we get ready to release. They will look something like this. There will be steps for importing the public keys and registering the repository before the apt-get update / apt-get install commands. 


## Executing commands in a Docker container

If you have a running Docker container, you can execute commands within the container from a host terminal.

To get the container ID run:
    
    $ docker ps

To start a bash terminal in the container run:

    $ docker exec -ti <container ID> /bin/bash

Now you can run commands as though you are running them at the terminal inside the container.

Example of how you could read the contents of the error log in the terminal window:

    $ docker exec -ti d6b75213ef80 /bin/bash root@d6b75213ef80:/# cat /var/opt/mssql/log/errorlog

### Copying files from a Docker container

To copy a file out of the container you could do something like this:
    
    $ docker cp <container ID>:<container path> <host path>

Example:
    
    $ docker cp d6b75213ef80:/var/opt/mssql/log/errorlog /tmp/errorlog

To copy a file in to the container you could do something like this:
    
    $ docker cp <host path> <container ID>:<container path>

Example:
    
    $ docker cp /tmp/mydb.mdf d6b75213ef80:/var/opt/mssql/data

## Accessing the Log Files
    
The SQL Server engine logs to the /var/opt/mssql/log/errorlog file in both the Linux and Docker installations. You need to be in ‘superuser’ mode to browse this directory.

The installer logs here: /var/opt/mssql/setup-<a time stamp representing time of install>
You can browse the errorlog files with any UTF-16 compatible tool like ‘vim’ or ‘cat’ like this: 

    $ cat errorlog

If you prefer, you can also convert the files to UTF-8 to read them with ‘more’ or ‘less’ with the following command:
    
    $ iconv –f UTF-16LE –t UTF-8 <errorlog> -o <output errorlog file>

## Extended Events

 Extended events can be queried via a SQL command.  More information about extended events can be found [here](https://technet.microsoft.com/en-us/library/bb630282.aspx):

## Sharing SQL Server on Linux Crash Dumps 

update for public preview

## Common Issues

1. Port 1433 conflicts when using the Docker image and SQL Server on Linux simultaneously

 When trying to run the SQL Server Docker image in parallel to SQL Server running on Ubuntu, check for the port number that it is running under. If it tries to run on the same port number, it will throw the following error: “failed to create endpoint <container name> on network bridge. Error starting proxy: listen tcp 0.0.0.0:1433 bind: address already in use.” This can also happen when running two Docker containers under the same port number.

2. ERROR: Hostname must be 15 characters or less

 This is a known-issue that happens whenever the name of the machine that is trying to install the SQL Server Debian package is longer than 15 characters. There are currently no workarounds other than changing the name of the machine. One way to achieve this is by editing the hostname file and rebooting the machine. The following [website guide](http://www.cyberciti.biz/faq/ubuntu-change-hostname-command/) explains this in detail.

3. Resetting the system administration (SA) password

 If you have forgotten the system administrator (SA) password or need to reset it for some other reason please follow these steps.

 > [!NOTE]
 > Following these steps will stop the SQL Server service temporarily.

 Log into the host terminal and run the following commands:

        $ systemctl stop mssql-server.service
        $ cd /var/opt/mssql
        $ sudo chsh mssql -s /bin/bash
        $ su mssql
        $ /opt/mssql/bin/sqlservr --setup --sa-password <password>
        $ exit
        $ systemctl start mssql-server.service

4. Using special characters in password

If you use some characters in the SQL Server login password you may need to escape them when using them in the Linux terminal.

For example, if you create your SA account password as follows:

    $ sudo ACCEPT_EULA=Y MSSQL_SERVER_SA_PASSWORD=Test$$ apt-get install mssql-server -y

You will need to escape the $ anytime using the backslash character you are using it in a terminal command/shell script:

Does not work:

    $ sqlcmd -S myserfver -U sa -P Test$$

Works:

    $ sqlcmd -S myserfver -U sa -P Test\$\$

Resources:

[Special characters](http://tldp.org/LDP/abs/html/special-chars.html)

[Escaping](http://tldp.org/LDP/abs/html/escapingsection.html)
