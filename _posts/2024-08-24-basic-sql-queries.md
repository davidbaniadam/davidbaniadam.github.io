---
layout: post
title: "Resource Governor"
date: 2024-08-24 10:00:00 +0000
show_date: false
categories: blog
permalink: resource-governor.html
published: true
author: "David Baniadam"
---

# Introduction

Resource Governor is an Enterprise-only feature[^1] and can help solve some problems, including performance problems. Some examples include: 
* Limit the amount of resources maintenance jobs (index maintenance and integrity checks) can use to avoid performance problems for databases that need to be available 24/7. 
* Solving Resource Semaphore waits. 
* On an instance with multiple databases from different departments that have to compete for the resource and when few users use all the computing resources on the server. 
* Testing before downsizing a server. 

# Resource Governor in SSMS
Seeing Resource Governor in SSMS might help with understanding the concepts: 

![SSMS-Resource-Governor](/images/ssms-resource-governor-1.png)  
*Figure 1: Resource Governor in SSMS*

The red cross over the icon of Resource Governor means that is is currently disabled. But disabled only means that the default configurations are being used. As can be seen in Figure 1, Resource Governor consists of Resource Pools and by default there are two "System Resource Pools", the *default pool* and the *internal pool* (note: when we refer to e.g. the default pool we are refering to the default *resource* pool). 

**Example for a Resource Pool**: All the CPU time on a given server is 100% of the CPU resources, so we can define a (user-defined) Resource Pool that consist of 50% of the CPU time. If a user is put into this Pool, then he cannot consume more that 50% of the CPU time on the server[^2] 

By default all user request end up in the *default pool*. The *internal pool* is for system tasks (e.g. Checkpoint, Lazy Writer) and cannot be modified in any way. The *default pool* cannot be dropped, but it can be altered. If we right click the *default pool* and choose "Properties" a window is opened, where the top part is shown in Figure 2

![SSMS-Resource-Governor](/images/ssms-properties.png)  
*Figure 2: properties of the default pool*

If we script out the default pool as "ALTER" (see Figure 3) then we see even more details that are not shown in the GUI. 

![SSMS-Resource-Governor](/images/ssms-resource-governor-2.png)  
*Figure 3: script out the default pool*

```sql
USE [master]
GO

ALTER RESOURCE POOL [default] WITH(
		MIN_CPU_PERCENT=0, 
		MAX_CPU_PERCENT=100, 
		MIN_MEMORY_PERCENT=0, 
		MAX_MEMORY_PERCENT=100, 
		CAP_CPU_PERCENT=100, 
		AFFINITY SCHEDULER = AUTO, 
		MIN_IOPS_PER_VOLUME=0, 
		MAX_IOPS_PER_VOLUME=0)
GO
```
We can create a user-defined Resource Pool as e.g.:

```sql
USE master;
GO
 
CREATE RESOURCE POOL UD_ResourcePool
WITH(
MAX_CPU_PERCENT = 50,
MAX_MEMORY_PERCENT = 50
);
GO
```


To read more about Resource Pools, Workload Groups and Classification see the [documentation](https://learn.microsoft.com/en-us/sql/relational-databases/resource-governor/resource-governor?view=sql-server-ver16#resource-concepts) 

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





[^1]: See https://learn.microsoft.com/en-us/sql/sql-server/editions-and-components-of-sql-server-2022?view=sql-server-ver16&preserve-view=true#RDBMSSP
[^2]: The description in the example requires CAP_CPU_PERCENT. 
