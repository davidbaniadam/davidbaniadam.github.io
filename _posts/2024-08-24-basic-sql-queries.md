---
layout: post
title: "Resource Governor Part 1"
date: 2024-08-24 10:00:00 +0000
categories: blog
---


Resource Governor is an Enterprise-only feature[^1] and can help solve some performance problems. Some exampoles include: 
* Limit the amount of resources maintenance jobs (index maintenance and integrity checks) can use to avoid performance problems for databases that need to be available 24/7. 
* On an instance with multiple databases from different departments that have to compete for the resource and when few users use all the computing resources on the server. 
* Solving Resource Semaphore waits. 
* Testing before downsizing a server. 

In these series we talk more about some of the above cases along with the code to implement the solution.

One should understand these three concepts: Resource Pools, Workload Groups and Classification; for more details see the official documentation for an explanation of these terms: [Resource Governor - SQL Server | Microsoft Learn](https://learn.microsoft.com/en-us/sql/relational-databases/resource-governor/resource-governor?view=sql-server-ver16#resource-concepts).  





## SELECT Query

The `SELECT` statement 

```sql
SELECT * FROM Table;
```

[^1]: See [Editions and supported features of SQL Server 2022 - SQL Server | Microsoft Learn.](https://learn.microsoft.com/en-us/sql/sql-server/editions-and-components-of-sql-server-2022?view=sql-server-ver16&preserve-view=true#RDBMSSP) 
