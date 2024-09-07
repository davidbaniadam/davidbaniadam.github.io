---
layout: post
title: "Resource Governor"
date: 2024-08-24 10:00:00 +0000
categories: blog
permalink: resource-governor.html
published: true
---

# Introduction

Resource Governor is an Enterprise-only feature[^1] and can help solve some performance problems. Some exampoles include: 
* Limit the amount of resources maintenance jobs (index maintenance and integrity checks) can use to avoid performance problems for databases that need to be available 24/7. 
* Solving Resource Semaphore waits. 
* On an instance with multiple databases from different departments that have to compete for the resource and when few users use all the computing resources on the server. 
* Testing before downsizing a server. 

# Resource Governor in SSMS
Seeing Resource Governor in SSMS might help with understanding the concepts: 

![SSMS-Resource-Governor](/images/ssms-resource-governor-1.png)

*Figure 1: Resource Governor in SSMS*

As can be seen in Figure 1, Resource Governor consists of Resource Pools. 

# Code Example
A code example will help in understanding how Resource Govornor works. 

```sql 
USE master;
GO
 
CREATE RESOURCE POOL UD_ResourcePool
WITH (
      MAX_CPU_PERCENT = 50,
      MAX_MEMORY_PERCENT = 50
      /*
      there is also MAX_IOPS_PER_VOLUME = 0
      */
      );
GO
 
CREATE WORKLOAD GROUP UD_WorkloadGroup
USING UD_ResourcePool;
GO
 
CREATE FUNCTION dbo.RG_Classifier()
RETURNS SYSNAME
WITH SCHEMABINDING
AS
BEGIN
    DECLARE @group_name AS SYSNAME;
 
    IF SUSER_SNAME() = 'domain\user'
        SET @group_name = 'UD_WorkloadGroup';
    ELSE
        SET @group_name = 'default';
 
    RETURN @group_name;
END;
GO


```

One should understand these three concepts: Resource Pools, Workload Groups and Classification; for more details see the official documentation for an explanation of these terms: [Resource Governor - SQL Server | Microsoft Learn](https://learn.microsoft.com/en-us/sql/relational-databases/resource-governor/resource-governor?view=sql-server-ver16#resource-concepts).  



[^1]: See [Editions and supported features of SQL Server 2022 - SQL Server | Microsoft Learn.](https://learn.microsoft.com/en-us/sql/sql-server/editions-and-components-of-sql-server-2022?view=sql-server-ver16&preserve-view=true#RDBMSSP) 
