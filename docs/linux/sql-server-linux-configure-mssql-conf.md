---
# required metadata

title: Configure SQL Server settings on Linux | Microsoft Docs
description: This topic describes how to use the mssql-conf tool to  configure SQL Server vNext settings on Linux.
author: luisbosquez 
ms.author: lbosq 
manager: jhubbard
ms.date: 04/25/2017
ms.topic: article
ms.prod: sql-linux
ms.technology: database-engine
ms.assetid: 06798dff-65c7-43e0-9ab3-ffb23374b322

# optional metadata

# keywords: ""
# ROBOTS: ""
# audience: ""
# ms.devlang: ""
# ms.reviewer: ""
# ms.suite: ""
# ms.tgt_pltfrm: ""
ms.custom: H1Hack27Feb2017

---
# Configure SQL Server on Linux with the mssql-conf tool
**mssql-conf** is a configuration script that installs with SQL Server vNext CTP 2.0 for Red Hat Enterprise Linux, SUSE Linux Enterprise Server, and Ubuntu. You can use this utility to set the following parameters:

- [TCP port](#tcpport): Change the port where SQL Server will listen for connections.
- [Default data directory](#datadir): Change the directory where the new SQL Server database data files (.mdf).
- [Default log directory](#datadir): Changes the directory where the new SQL Server database log (.ldf) files are created.
- [Default dump directory](#dumpdir): Change the directory where SQL Server will deposit the memory dumps and other troubleshooting files by default.
- [Default backup directory](#backupdir): Change the directory where SQL Server will send the backup files by default. 
- [Mini and full dump preferences](#coredump): Specify whether to generate both mini dumps and full dumps.
- [Core dump type](#coredump): Choose the type of dump memory dump file to collect.
- [Set traceflags](#traceflags): Set the traceflags that the service is going to use.
- [Set collation](#collation): Set a new collation for SQL Server on Linux.

The following sections show examples of how to use mssql-conf for each of these scenarios.

> [!TIP]
> These examples run mssql-conf by specify the full path: `/opt/mssql/bin/mssql-conf`. If you choose to navigate to that path instead, run mssql-conf in the context of the current directory: `./mssql-conf`.

## <a id="tcpport"></a> Change the TCP port

This option will let you change the TCP port where SQL Server will listen for connections. By default, this port is set to 1433. To change the port, run the following commands:

1. Run the mssql-conf script as root with the "set" command for "tcpport":

   ```bash
   sudo /opt/mssql/bin/mssql-conf set tcpport <new_tcp_port>
   ```

2. Restart the SQL Server service as instructed by the configuration utility:

   ```bash
   sudo systemctl restart mssql-server
   ```

3. When connecting to SQL Server now, you will need to specify the port with a comma (,) after the hostname or IP address. For example, to connect with SQLCMD, you would use the following command:

   ```bash
   sqlcmd -S localhost,<new_tcp_port> -U test -P test
   ```

## <a id="datadir"></a> Change the default data or log directory location

This option will let you change the location where the new database and log files are created. By default, this location is /var/opt/mssql/data. To achieve this, follow these steps:

1. Create the directory where the new database's data and log files will reside. For example, we will use /tmp/data:  

   ```bash
   sudo mkdir /tmp/data
   ```

2. Change the owner and group of the directory to the "mssql" user:

   ```bash
   sudo chown mssql /tmp/data
   sudo chgrp mssql /tmp/data
   ```

3. Use mssql-conf to change the default data directory with the "set" command:

   ```bash
   sudo /opt/mssql/bin/mssql-conf set defaultdatadir /tmp/data
   ```

4. Restart the SQL Server service as instructed by the configuration utility:

   ```bash
   sudo systemctl restart mssql-server
   ```

5. Now all the database files for the new databases created will be stored in this new location. If you would like to change the location of the log (.ldf) files of the new databases, you can use the following "set" command:

   ```bash
   sudo /opt/mssql/bin/mssql-conf set defaultlogdir /tmp/log
   ```

6. This command also assumes that a /tmp/log directory exists, and that it is under the user and group "mssql".

## <a id="dumpdir"></a> Change the default dump directory location

This option will let you change the default location where the memory and SQL dumps are generated whenever there is a crash. By default, these files are generated in /var/opt/mssql/log.

To set up this new location, use the following commands:

1. Create the directory where the dump files will reside. For example, we will use /tmp/dump:  

   ```bash
   sudo mkdir /tmp/dump
   ```

2. Change the owner and group of the directory to the "mssql" user:

   ```bash
   sudo chown mssql /tmp/dump
   sudo chgrp mssql /tmp/dump
   ```

3. Use mssql-conf to change the default data directory with the "set" command:

   ```bash
   sudo /opt/mssql/bin/mssql-conf set defaultdumpdir /tmp/dump
   ```

4. Restart the SQL Server service as instructed by the configuration utility:

   ```bash
   sudo systemctl restart mssql-server
   ```

## <a id="backupdir"></a> Change the default backup directory location

This option will let you change the default location where the backup files are generated. By default, these files are generated in /var/opt/mssql/data.

To set up this new location, use the following commands:

1. Create the directory where the backup files will reside. For example, we will use /tmp/backup:  

   ```bash
   sudo mkdir /tmp/backup
   ```

2. Change the owner and group of the directory to the "mssql" user:

   ```bash
   sudo chown mssql /tmp/backup
   sudo chgrp mssql /tmp/backup
   ```

3. Use mssql-conf to change the default backup directory with the "set" command:

   ```bash
   sudo /opt/mssql/bin/mssql-conf set defaultbackupdir /tmp/backup
   ```

4. Restart the SQL Server service as instructed by the configuration utility:

   ```bash
   sudo systemctl restart mssql-server
   ```

## <a id="coredump"></a> Specify core dump settings

If an exception occurs in one of the SQL Server processes, SQL Server creates a memory dump. 

There are two options for controlling the type of memory dumps that SQL Server collects: **coredumptype** and **captureminiandfull**. These relate to the two phases of core dump capture. 

The first phase capture is controlled by the **coredumptype** setting, which determines the type of dump file generated during an exception. The second phase is enabled when the **captureminiandfull** setting. If **captureminiandfull** is set to true, the dump file specified by **coredumptype** is generated and a second mini dump is also generated. Setting **captureminiandfull** to false disables the second capture attempt.

1. Decide whether to capture both mini and full dumps with the **captureminiandfull** setting.

    ```bash
    sudo /opt/mssql/bin/mssql-conf set captureminiandfull <true or false>
    ```

    Default: **true**

2. Specify the type of dump file with the **coredumptype** setting.

    ```bash
    sudo /opt/mssql/bin/mssql-conf set coredumptype <dump_type>
    ```
    
    Default: **miniplus**

    The following table lists the possible **coredumptype** values.

    | Type | Description |
    |-----|-----|
    | **mini** | Mini is the smallest dump file type. It uses the Linux system information to determine threads and modules in the process. The dump contains only the host environment thread stacks and modules. It does not contain indirect memory references or globals. |
    | **miniplus** | MiniPlus is similar to mini, but it includes additional memory. It understands the internals of SQLPAL and the host environment, adding the following memory regions to the dump:</br></br> - Various globals</br> - All memory above 64TB</br> - All named regions found in **/proc/$pid/maps**</br> - Indirect memory from threads and stacks</br> - Thread information</br> - Associated Teb’s and Peb’s</br> - Module Information</br> - VMM and VAD tree |
    | **filtered** | Filtered uses a subtraction-based design where all memory in the process is included unless specifically excluded. The design understands the internals of SQLPAL and the host environment, excluding certain regions from the dump.
    | **full** | Full is a complete process dump that includes all regions located in **/proc/$pid/maps**. This is not controlled by **captureminiandfull** setting. |

## <a id="traceflags"></a> Enable/Disable traceflags

This option will let you enable or disable traceflags for the startup of the SQL Server service. To enable/disable a traceflag use the following commands:

1. Enable a traceflag using the following command. For example, for Traceflag 1234:  

   ```bash
   sudo /opt/mssql/bin/mssql-conf traceflag 1234 on
   ```

2. You can enable multiple traceflags by specifying them separately:

   ```bash
   sudo /opt/mssql/bin/mssql-conf traceflag 2345 3456 on
   ```

3. In a similar way, you can disable one or more enabled traceflags by specifying them and adding the "off" parameter:

   ```bash
   sudo /opt/mssql/bin/mssql-conf traceflag 1234 2345 3456 off
   ```

4. Restart the SQL Server service as instructed by the configuration utility for the changes to apply:

   ```bash
   sudo systemctl restart mssql-server
   ```

## <a id="collation"></a> Change the SQL Server collation

This option will let you change the collation value to any of the supported collations:


1. Run the "set-collation" option and follow the prompts:

   ```bash
   sudo /opt/mssql/bin/mssql-conf set-collation
   ```

2. The mssql-conf utility will try to restore the databases using the specified collation and restart the service. If there are any errors, it will roll-back the collation to the previous value.

For a list of supported collations, run the [sys.fn_helpcollations](../relational-databases/system-functions/sys-fn-helpcollations-transact-sql.md) function: `SELECT Name from sys.fn_helpcollations()`.

## Next steps

For other management tools and scenarios, see [Manage SQL Server on Linux](sql-server-linux-management-overview.md).
