---
# required metadata

title: Install SQL Server on Ubuntu | Microsoft Docs
description: 
author: rothja 
ms.author: jroth 
manager: jhubbard
ms.date: 02/02/2017
ms.topic: article
ms.prod: sql-linux
ms.technology: database-engine
ms.assetid: 31c8c92e-12fe-4728-9b95-4bc028250d85

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
# Install SQL Server on Ubuntu

This topic provides a walkthrough of how to install SQL Server vNext CTP 1.3 on Ubuntu 16.04 and 16.10.

> [!NOTE] 
> You need at least 3.25GB of memory to run SQL Server on Linux.
> SQL Server Engine has only been tested up to 256GB of memory at this time.

## Install SQL Server

To install the **mssql-server** Package on Ubuntu, follow these steps:

1. Import the public repository GPG keys:

   ```bash
   curl https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
   ```

3. Register the Microsoft SQL Server Ubuntu repository:

   ```bash
   curl https://packages.microsoft.com/config/ubuntu/16.04/mssql-server.list | sudo tee /etc/apt/sources.list.d/mssql-server.list
   ```

5. Run the following commands to install SQL Server:

   ```bash
   sudo apt-get update
   sudo apt-get install -y mssql-server
   ```

6. After the package installation finishes, run the configuration script and follow the prompts. Make sure to specify a strong password for the SA account (Minimum length 8 characters, including uppercase and lowercase letters, base 10 digits and/or non-alphanumeric symbols).
 
   ```bash
   sudo /opt/mssql/bin/sqlservr-setup
   ```

7. Once the configuration is done, verify that the service is running:

   ```bash
   systemctl status mssql-server
   ```

8. To allow remote connections, you may need to open the SQL Server TCP port on your firewall. The default SQL Server port is 1433.

## Upgrade SQL Server

To upgrade the **mssql-server** package on Ubuntu, follow these steps:

1. Update the apt-get repository lists:
   ```bash
   sudo apt-get update
   ```

2. Re-run the installation command, this will upgrade the specific mssql-server package:
   ```bash
   sudo apt-get install mssql-server
   ```

These commands will download the newest package and replace the binaries located under `/opt/mssql/`. The user generated databases and system databases will not be affected by this operation. 

## Uninstall SQL Server

To remove the **mssql-server** package on Ubuntu, follow these steps:

1. Run the `remove` command. This will delete the package and remove the files under `/opt/mssql/`. However, this command will not affect user-generated and system databases.
   ```bash
   sudo apt-get remove mssql-server
   ```

2. Removing the package will not delete the generated database files. If you want to delete the database files use the following command:
   ```bash
   sudo rm -rf /var/opt/mssql/
   ```


## Next steps

- [Install tools](sql-server-linux-setup-tools.md#ubuntu)

- If you already have SQL Server tools, [connect to the SQL Server](sql-server-linux-connect-and-query-sqlcmd.md).
