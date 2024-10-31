---
layout: post
title: "Debugging SQL Server Notes"
date: 2024-08-24 10:00:00 +0000
show_date: false
categories: blog
permalink: debugging-sql-server.html
published: false
author: "David Baniadam"
---

# Useful Resources


# Download Symbols
There are two ways do download symbols for offline use: Creating a stackdump or using a tool called 

## Use the symchk tool
If you do not have SQL Server installed on your computer with internet access then you can copy the Binn folder to your computer. In the command below the symbols will be downloaded to `C:\Symbols`. 

```powersehll
.\symchk.exe /r C:\sqlservr\Binn\ /s SRV*C:\Symbols*https://msdl.microsoft.com/download/symbols
```
## Create a Stackdump
Use `DBCC STACKDUMP` to create a stackdump and transfer this file to your computer with internet access. 

# Useful commands


```powershell
# Set location of symbols when working offline
.sympath C:\Symbols

# Display the call stack for all threads in the system. ~* means all threads and kv shows the stack with frame memory and related information. 
~*kv

# dusokat the call stack for current thread. 
k

# Set breakpoint
bp function_name

# show breakpoints: breakpoint list
bl

# breakpoint disable with id 0
db 0

# breakpoint clear with id 0
bc 0

# list modules verbose
lm v

# resume execution: go
g

# Break
Alt + Del

# group similar stacks
!uniqstack
```

links: https://techcommunity.microsoft.com/t5/sql-server-support-blog/intro-to-debugging-a-memory-dump/ba-p/316925
