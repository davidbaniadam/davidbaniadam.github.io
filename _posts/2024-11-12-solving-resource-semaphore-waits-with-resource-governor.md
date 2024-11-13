---
title: "Solving RESOURCE_SEMAPHORE Waits with Resource Governor"
date: 2024-10-29T12:24:30-05:00
published: false
categories:
  - blog
tags:
  - Resource Governor
---


# Solving Resource Semaphore waits  
The RESOURCE_SEMAPHORE wait type is one of those wait types that can cause performance disasters and the reason is that SQL Server cannot give the queries the memory they need so that they are able to run. A lot of advice on the internet suggests to increase the amount of RAM on a server. The objective of this article is to show that this advice is not always true and it depends and there are multiple potential causes for RESOURCE_SEMAPHORE waits. A simple equation can hopefully shed light on this point:  

$$
c = \frac{a}{b}
$$

where $b >0$. How can we increase the value of *c*? The answer is that we can either increase the value of *a* or we can decrease the value of *b*. Similary with RESOURCE_SEMAPHORE waits we need to understand what is causing these waits. 



As an example assume $a=128 GB$ and $b=0.25

This example hopefully illustrates that one should try to understand some relationships before blindly following some advice on the internet. 

## Memory Grants
In this section we wan to understand memory grants. Queries that need to sort data or perform hash operations need memory in order to run. But what is the maximum amount of memory a query can receive for memory grants (called MaxQueryMemory in the execution plan)? To simplify the discussion will assume that the "default" workload group is the only workload group (besides the internal workload group) and than the default resource pool is the only resource pool (besides the internal resource pool). There are at least two factors that determine MaxQueryMemory: The amount of memory on the box and the resource governor settings in the workload group. 

The exact formula is not documented, but "the amount of memory" could very well be committed_target_kb in sys.dm_os_sys_info. committed_target_kb cannot go above the "max server memory" server configuration setting and usually they are equal, but it is possible for committed_target_kb to become less than "max server memory" and the value is not fixed.  





that Going back to RESOURCE_SEMAPHORE waits we can say that sometimes we might have to add more memory to the server, but sometimes adding more memory does not solve the problem and we have to control the amount of memory a single query can be granted. 
