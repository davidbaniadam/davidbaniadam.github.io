---
layout: post
title: "Introduction to SQL Server Resource Governor"
date: 2024-08-24 10:00:00 +0000
show_date: false
categories: blog
permalink: resource-governor.html
published: true
author: "David Baniadam"
---

# Table of Contents
- [Introduction](#introduction)
- [Resource Governor In SSMS](#resource-governor-in-ssms)
	- [Resource Pools](#resource-pools)  
	- [Workload Groups and Classification](#workload-groups-and-classification)  
- [Code Example to Limit Resources for Maintenance Jobs](#code-example-to-limit-resources-for-maintenance-jobs)  
- [Solving Resource Semaphore waits](#solving-resource-semaphore-waits)
- [Monitoring Resource Usage in Different Pools](#monitoring-resource-usage-in-different-pools)
- [How to Limit IOPS](#how-to-limit-iops)

# Introduction

Resource Governor is an Enterprise-only feature[^1] and can help solve some interesting problems, including performance problems. Some examples include: 
* Limit the amount of resources maintenance jobs (index maintenance and integrity checks) can use to avoid performance problems for databases that need to be available 24/7. 
* Solving Resource Semaphore waits. 
* Limit the amount of resources a user/group can use. 
* Monitoring how many resources different users/departments/apps etc. use. 
* Testing before downsizing a server.

It can be quite difficult to learn Resource Governor by reading the documentation. This post tries to make understading Resource Governor easier. 


# Resource Governor In SSMS
Seeing resource-governor-in-ssms might help with understanding the concepts: 

![SSMS-Resource-Governor](/images/ssms-resource-governor-1.png)  
*Figure 1: resource-governor-in-ssms*

The red cross over the icon of Resource Governor means that is is currently disabled, but "disabled" only means that the default configurations are being used. As can be seen in Figure 1, Resource Governor consists of Resource Pools and by default there are two "System Resource Pools": The *default pool* and the *internal pool* (note: when we refer to e.g. the default pool we are refering to the default *resource* pool). 

## Resource Pools
What is a Resource Pool? An example will perhaps help:   
> **Example for Resource Pool**: All the CPU time on a given server is 100% of the CPU resources, so we can define a (user-defined) Resource Pool that consist of 50% of the CPU time. If a user is put into this Pool, then he cannot consume more that 50% of the CPU time on the server[^2] 

By default all user request end up in the *default pool*. The *internal pool* is for system tasks (e.g. Checkpoint, Lazy Writer) and cannot be modified. The *default pool* cannot be dropped, but it can be altered. If we right click the *default pool* and choose "Properties" a window is opened, see Figure 2

![SSMS-Resource-Governor](/images/ssms-properties.png)  
*Figure 2: properties of the default pool*

The top part of Figure 2 shows the properties of the two system resource pools. If we script out the default pool as "ALTER" (see Figure 3) then we see even more details that are not shown in the GUI. 

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
		MAX_IOPS_PER_VOLUME=0);
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
## Workload Groups and Classification
The next terms we need to understand are *Workload Groups* and *Classification*. A session is clasified into a Workload Group which is then put into a resource pull. The relationship between a Resource Pool and a Workload Group is 1-to-many, i.e. multiple workload groups can be put a resource pool. The workload group has also some settings that are quite important. We can also script out the default workload group (see also the bottom part of Figure 2):

```sql
USE [master]
GO

ALTER WORKLOAD GROUP [default] 
	WITH(	
		GROUP_MAX_REQUESTS=0, 
		IMPORTANCE=Medium, 
		REQUEST_MAX_CPU_TIME_SEC=0, 
		REQUEST_MAX_MEMORY_GRANT_PERCENT=25, 
		REQUEST_MEMORY_GRANT_TIMEOUT_SEC=0, 
		MAX_DOP=0
		);
GO
```

Classification is the process of putting sessions into a workload group and is done by writing a user-defined function in the master database. In the absense of such a function all user sessions are mapped into the default workload group. Some system function that can be used in writing the classifier function are: 

```sql
SELECT
ORIGINAL_LOGIN(), 
APP_NAME(), 
SUSER_SNAME(), 
HOST_NAME(), 
PROGRAM_NAME(), 
IS_SRVROLEMEMBER('sysadmin'), 
IS_MEMBER('db_owner'),
IS_MEMBER('domain\group') as [AD group],
CONNECTIONPROPERTY('auth_scheme');

```
In a sense a workload group is an extra layer of complexity 

To read more about Resource Pools, Workload Groups and Classification see the [documentation](https://learn.microsoft.com/en-us/sql/relational-databases/resource-governor/resource-governor?view=sql-server-ver16#resource-concepts) 

# Code Example to Limit Resources for Maintenance Jobs
A code example will help in understanding how Resource Govornor works. 

```sql 
USE master;
GO
 
CREATE RESOURCE POOL UD_ResourcePool
WITH (
      MAX_CPU_PERCENT = 50,
      MAX_MEMORY_PERCENT = 50
      );
GO
 
CREATE WORKLOAD GROUP UD_WorkloadGroup
USING UD_ResourcePool;
GO
 
CREATE FUNCTION dbo.Classifier()
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

-- Register the classifier function with Resource Governor
ALTER RESOURCE GOVERNOR WITH (CLASSIFIER_FUNCTION = dbo.Classifier);
ALTER RESOURCE GOVERNOR RECONFIGURE;
GO

```

# Solving Resource Semaphore waits  
The RESOURCE_SEMAPHORE wait type is one of those wait types that can cause performance disasters. As we will see there are multiple factors (variables) that can be the source of RESOURCE_SEMAPHORE waits. A simple equation can hopefully shed light on this point:  

$$
c = \frac{a}{b}
$$

How can we increase the value of *c*? The answer is that we can either increase the value of *a* or we can decrease the value of *b*. This example hopefully illustrates that one should try to understand some relationships before blindly following some advice on the internet. 

## Memory Grants
In this section we wan to understand memory grants. Queries that need to sort data or perform hash operations need memory in order to run. But what is the maximum amount of memory a query can receive for memory grants (called MaxQueryMemory in the execution plan)? To simplify the discussion will assume that the "default" workload group is the only workload group (besides the internal workload group) and than the default resource pool is the only resource pool (besides the internal resource pool). There are at least two factors that determine MaxQueryMemory: The amount of memory on the box and the resource governor settings in the workload group. 

The exact formula is not documented, but "the amount of memory" could very well be committed_target_kb in sys.dm_os_sys_info. committed_target_kb cannot go above the "max server memory" server configuration setting and usually they are equal, but it is possible for committed_target_kb to become less than "max server memory" and the value is not fixed.  





that Going back to RESOURCE_SEMAPHORE waits we can say that sometimes we might have to add more memory to the server, but sometimes adding more memory does not solve the problem and we have to control the amount of memory a single query can be granted. 

# Monitoring Resource Usage in Different Pools  

Comming soon.

# How to Limit IOPS  

Comming soon.



[^1]: See https://learn.microsoft.com/en-us/sql/sql-server/editions-and-components-of-sql-server-2022?view=sql-server-ver16&preserve-view=true#RDBMSSP
[^2]: The description in the example requires CAP_CPU_PERCENT. 
