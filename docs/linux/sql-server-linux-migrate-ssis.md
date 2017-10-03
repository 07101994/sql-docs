---
title: Extract, transform, and load data on Linux with SSIS | Microsoft Docs
description: 
author: leolimsft 
ms.author: lle 
ms.reviewer: douglasl
manager: craigg
ms.date: 10/02/2017
ms.topic: article
ms.prod: sql-linux
ms.technology: database-engine
ms.assetid: 9dab69c7-73af-4340-aef0-de057356b791
---
# Extract, transform, and load data on Linux with SSIS

[!INCLUDE[tsql-appliesto-sslinux-only](../includes/tsql-appliesto-sslinux-only.md)]

This topic describes how to run SQL Server Integration Services (SSIS) packages on Linux. SSIS solves complex data integration problems by extracting data from multiple sources and formats, transforming and cleansing the data, and loading the data into multiple destinations. 

SSIS packages running on Linux can connect to Microsoft SQL Server running on Windows on-premises or in the cloud, on Linux, or in Docker. They can also connect to Azure SQL Database, Azure SQL Data Warehouse, ODBC data sources, flat files, and other data sources including ADO.NET sources, XML files, and OData services.

For more info about the capabilities of SSIS, see [SQL Server Integration Services](../integration-services/sql-server-integration-services.md).

## Prerequisites

To run SSIS packages on a Linux computer, first you have to install SQL Server Integration Services. SSIS is not included in the installation of SQL Server for Linux computers. For installation instructions, see [Install SQL Server Integration Services](sql-server-linux-setup-ssis.md).

You also have to have a Windows computer to create and maintain packages. The SSIS design and management tools are Windows applications that are not currently available for Linux computers. 

## Run an SSIS package

To run an SSIS package on a Linux computer, do the following things:

1.  Copy the SSIS package to the Linux computer.
2.  Run the following command:
    ```
    $ dtexec /F \<package name \> /DE <protection password>
    ```

## Other common SSIS tasks

-   **Design packages**.

    -   **Connect to ODBC data sources**. With SSIS on Linux CTP 2.1 Refresh and later, SSIS packages can use ODBC connections on Linux. This functionality has been tested with the SQL Server and the MySQL ODBC drivers, but is also expected to work with any Unicode ODBC driver that observes the ODBC specification. At design time, you can provide either a DSN or a connection string to connect to the ODBC data; you can also use Windows authentication. For more info, see the [blog post announcing ODBC support on Linux](https://blogs.msdn.microsoft.com/ssis/2017/06/16/odbc-is-supported-in-ssis-on-linux-ssis-helsinki-ctp2-1-refresh/).

    -   **Paths**. Provide Windows-style paths in your SSIS packages. SSIS on Linux does not support Linux-style paths, but maps Windows-style paths to Linux-style paths at run time. Then, for example, SSIS on Linux maps the Windows-style path `C:\test` to the Linux-style path `/test`.

-   **Deploy packages**. You can only store packages in the file system on Linux in this release. The SSIS Catalog database and the legacy SSIS service are not available on Linux for package deployment and storage.

-   **Schedule packages**. You can use Linux system scheduling tools such as `cron` to schedule packages. You can't use SQL Agent on Linux to schedule package execution in this release. For more info, see [Schedule SSIS packages on Linux with cron](sql-server-linux-schedule-ssis-packages.md).

## Limitations and known issues

### General limitations and known issues

The following features are not supported in this release of SSIS on Linux:
  - SSIS Catalog database
  - Scheduled package execution by SQL Agent
  - Windows Authentication
  - Third-party components
  - Change Data Capture (CDC)
  - SSIS Scale Out
  - Azure Feature Pack for SSIS
  - Hadoop and HDFS support
  - Microsoft Connector for SAP BW

For other limitations and known issues with SSIS on Linux, see the [Release Notes](sql-server-linux-release-notes.md#ssis).

### <a name="components"></a> Supported and unsupported components

The following built-in Integration Services components are supported on Linux. Some of them have limitations on the Linux platform, as described in the following tables.

Built-in components that are not listed here are not supported on Linux.

#### Supported control flow tasks
- Bulk Insert Task
- Data Flow Task
- Data Profiling Task
- Execute SQL Task
- Execute T-SQL Statement Task
- Expression Task
- FTP Task
- Web Service Task
- XML Task

#### Control flow tasks supported with limitations

| Task | Limitations |
|------------|---|
| Execute Process task | Only supports in-process mode. |
| File System task | The *Move directory* and *Set file attributes* actions are not supported. |
| Script task | Only supports standard .NET Framework APIs. |
| Send Mail task | Only supports anonymous user mode. |
| Transfer Database task | UNC paths are not supported. |
| | |

#### Supported control flow containers
- Sequence Container
- For Loop Container
- Foreach Loop Container

#### Supported data flow sources and destinations
- Raw File source and destination
- XML Source

#### Data flow sources and destinations supported with limitations

| Component | Limitations |
|------------|---|
| ADO.NET source and destination | Only support the SQLClient data provider. |
| Flat File source and destination | Only support Windows-style file paths, to which the default path mapping rule is applied. For example `D:\home\ssis\travel.csv` becomes `/home/ssis/travel.csv`. |
| OData source | Only supports Basic authentication. |
| ODBC source and destination | Supports 64-bit Unicode ODBC drivers on Linux. Depends on the UnixODBC driver manager on Linux. |
| OLE DB source and destination | Only support SQL Server Native Client 11.0 and Microsoft OLE DB Provider for SQL Server. |
| | |

#### Supported data flow transformations
- Aggregate
- Audit
- Balanced Data Distributor
- Character Map
- Conditional Split
- Copy Column
- Data Conversion
- Derived Column
- Export Column
- Fuzzy Grouping
- Fuzzy Lookup
- Import Column
- Lookup
- Merge
- Merge Join
- Multicast
- Pivot
- Row Count
- Slowly Changing Dimension
- Sort
- Term Lookup
- Union All
- Unpivot

#### Data flow transformations supported with limitations

| Component | Limitations |
|------------|---|
| OLE DB Command transformation | Same limitations as the OLE DB source and destination. |
| Script component | Only supports standard .NET Framework APIs. |
| | |

### Supported and unsupported log providers
All the built-in SSIS log providers are supported on Linux except the Windows Event Log provider.

The SQL Server log provider supports only SQL Authentication; it does not support Windows Authentication.

The SSIS log providers for Text files, for XML files, and for SQL Server Profiler write their output to a file that you specify. The following considerations apply to the file path:
-   If you don't provide a path, the log provider writes to the current directory of the host. If the current user doesn't have permission to write to the current directory of the host, the log provider raises an error.
-   You can't use an environment variable in a file path. If you specify an environment variable, the literal text that you specify appears in the file path. For example, if you specify `%TMP%/log.txt`, the log  provider appends the literal text `/%TMP%/log.txt` to the current host directory.

## More info about SSIS on Linux

For more info about SSIS on Linux, see the following blog posts:

-   [SSIS on Linux is available in SQL Server 2017 CTP2.1](https://blogs.msdn.microsoft.com/ssis/2017/05/17/ssis-helsinki-is-available-in-sql-server-vnext-ctp2-1/)
-   [ODBC is supported in SSIS on Linux (SQL Server 2017 CTP 2.1 refresh)](https://blogs.msdn.microsoft.com/ssis/2017/06/16/odbc-is-supported-in-ssis-on-linux-ssis-helsinki-ctp2-1-refresh/)

## More info about SSIS

Microsoft SQL Server Integration Services (SSIS) is a platform for building high-performance data integration solutions, including extraction, transformation, and loading (ETL) packages for data warehousing. For more info about SSIS, see [SQL Server Integration Services](/sql/integration-services/sql-server-integration-services).

SSIS includes the following features:
- Graphical tools and wizards for building and debugging packages on Windows
- A variety of tasks for performing workflow functions such as FTP operations, executing SQL statements, and sending e-mail messages
- A variety of data sources and destinations for extracting and loading data
- A variety of transformations for cleaning, aggregating, merging, and copying data
- Application programming interfaces (APIs) for extending SSIS with your own custom scripts and components

To get started with SSIS, download the latest version of [SQL Server Data Tools (SSDT)](../integration-services/ssis-how-to-create-an-etl-package.md).

## See also
- [Learn more about SQL Server Integration Services](../integration-services/sql-server-integration-services.md)
- [SQL Server Integration Services (SSIS) Development and Management Tools](../integration-services/integration-services-ssis-development-and-management-tools.md)
- [SQL Server Integration Services Tutorials](../integration-services/integration-services-tutorials.md)
