---
# required metadata

title: Overview of development for SQL Server on Linux - SQL Server vNext | Microsoft Docs
description: 
author: sanagama 
ms.author: sanagama 
manager: jhubbard
ms.date: 11/16/2016
ms.topic: article
ms.prod: sql-linux
ms.technology: database-engine
ms.assetid: 758cb738-b018-465b-9ab0-59a24b892e66

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
# Develop applications for SQL Server on Linux

You can create applications that connect to and use SQL Server vNext CTP 1.1 on Linux from a variety of programming languages, such as C#, Java, Node.js, PHP, Python, Ruby, and C++. You can also use popular web frameworks and Object Relational Mapping (ORM) frameworks.

> [!TIP]
> These same development options also enable you to target SQL Server on other platforms. Applications can target SQL Server running on-premises or in the cloud, on Linux, Windows, or Docker on macOS. Or you can target Azure SQL Database and Azure SQL Data Warehouse.

## Try the tutorials
The best way to get started and build applications with SQL Server is to try it out for yourself.

- Browse to [Getting Started Tutorials](http://aka.ms/sqldev).
- Select your language and development platform.
- Try the code samples.

> [!TIP]
> If you want to develop for SQL Server vNext CTP 1.1 on Docker, take a look at the **macOS** tutorials.

## Create new applications
If you're creating a new application, take a look at a list of the [Connectivity libraries](sql-server-linux-develop-connectivity-libraries.md) for a summary of the connectors and popular frameworks available for various programming languages.

## Use existing applications
If you have an existing database application, you can simply change its connection string to target SQL Server vNext CTP 1.1 on Linux. Make sure to read about the [Known Issues](sql-server-linux-release-notes.md) in SQL Server vNext CTP 1.1 on Linux.

## Use existing SQL tools on Windows with SQL Server on Linux
Tools that currently run on Windows such as SSMS, SSDT, and PowerShell, also work with SQL Server vNext CTP 1.1 on Linux. Although they do not run natively on Linux, you can still manage remote SQL Server instances on Linux. 

See the following topics for more information:

- [SQL Server Management Studio (SSMS)](sql-server-linux-develop-use-ssms.md)
- [SQL Server Data Tools (SSDT)](sql-server-linux-develop-use-ssdt.md)
- [SQL PowerShell](sql-server-linux-manage-powershell.md)

> [!Note] 
> Make sure that you are using the latest versions of these tools for the best experience.

## Use new SQL tools for Linux
You can use the new [mssql extension](https://aka.ms/mssql-marketplace) for [Visual Studio Code](https://code.visualstudio.com) on Linux, macOS, and Windows. For a step-by-step walkthrough, see the following tutorial:
- [Use Visual Studio Code](sql-server-linux-develop-use-vscode.md)

You can also use new command-line tools that are native for Linux. These tools include the following:
- [sqlcmd](sql-server-linux-connect-and-query-sqlcmd.md)
- [bcp](sql-server-linux-migrate-bcp.md)
- [sqlpackage](sql-server-linux-migrate-sqlpackage.md)
- [mssql-conf](sql-server-linux-configure-mssql-conf.md)

## Next steps
To install SQL Server vNext on Linux and get started, see [Get started with SQL Server on Linux](sql-server-linux-get-started-tutorial.md).
