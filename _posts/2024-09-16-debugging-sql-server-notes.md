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
There are two ways do download symbols for offline use. If you do not have SQL Server installed on your computer with internet access then you can copy the Binn folder to your computer. In the command below the symbols will be downloaded to `C:\Symbols`. 

```powersehll
.\symchk.exe /r C:\sqlservr\Binn\ /s SRV*C:\Symbols*https://msdl.microsoft.com/download/symbols
```

# Useful commands

Set location of symbols when working offline:
```powershell
.sympath C:\Symbols
```


