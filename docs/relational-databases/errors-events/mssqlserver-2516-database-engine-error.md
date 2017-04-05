---
title: "MSSQLSERVER_2516_deleted | Microsoft Docs"
ms.custom: ""
ms.date: "04/04/2017"
ms.prod: "sql-server-2016"
ms.reviewer: ""
ms.suite: ""
ms.technology: 
  - "database-engine"
ms.tgt_pltfrm: ""
ms.topic: "language-reference"
helpviewer_keywords: 
  - "2516 (Database Engine error)"
ms.assetid: 79ed14b5-f53c-40c6-8334-8a083f732207
caps.latest.revision: 19
author: "BYHAM"
ms.author: "rickbyh"
manager: "jhubbard"
---
# MSSQLSERVER_2516_deleted
  
## Details  
  
|||  
|-|-|  
|Product Name|SQL Server|  
|Event ID|2516|  
|Event Source|MSSQLSERVER|  
|Component|SQLEngine|  
|Symbolic Name|DBCC_REPAIR_DIFF_MAP_INVALIDATED|  
|Message Text|Repair has invalidated the differential bitmap for database NAME. The differential backup chain is broken. You must perform a full database backup before you can perform a differential backup.|  
  
## Explanation  
This informational message is returned when error MSSQLEngine_2515 is repaired.  
  
## User Action  
Perform a full database backup.  
  
## See Also  
[Create a Full Database Backup &#40;SQL Server&#41;](../Topic/Create%20a%20Full%20Database%20Backup%20(SQL%20Server).md)  
  
