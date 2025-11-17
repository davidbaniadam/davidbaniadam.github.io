---
title: "Using DISKSPD to Benchmark Storage"
date: 2025-11-15T12:24:30-05:00
published: false
categories:
  - blog
tags:
  - Diskspd
---

## Some useful Microsoft Learn pages
- Getting started page: https://learn.microsoft.com/en-us/previous-versions/azure/azure-local/manage/diskspd-overview
- Benchmark a disk: https://learn.microsoft.com/en-us/azure/virtual-machines/disks-benchmarks
- Design for Performance: https://learn.microsoft.com/en-us/azure/virtual-machines/premium-storage-performance

These pages also seems interesting: 
- https://techcommunity.microsoft.com/blog/filecab/windows-server-2025-storage-performance-with-diskspd/4167713
- https://learn.microsoft.com/en-us/sharepoint/administration/storage-and-sql-server-capacity-planning-and-configuration

  ## Example for a DW workload
  
```powershell
# Define the ZIP URL and the full path to save the file, including the filename
$zipName = "DiskSpd.zip"
$zipPath = "C:\DiskSpd"
$zipFullName = Join-Path $zipPath $zipName
$zipUrl = "https://github.com/microsoft/diskspd/releases/latest/download/" +$zipName

# Ensure the target directory exists, if not then create
if (-Not (Test-Path $zipPath)) {
New-Item -Path $zipPath -ItemType Directory | Out-Null
}
# Download and expand the ZIP file
Invoke-RestMethod -Uri $zipUrl -OutFile $zipFullName
Expand-Archive -Path $zipFullName -DestinationPath $zipPath;

# get the help
C:\DiskSpd\amd64\diskspd.exe /?

C:\DiskSpd\amd64\diskspd.exe -t1 -o32 -b512k -w0 -d120 -c10G  C:\temp\IO.dat > C:\temp\test01.txt;
remove-item -path  C:\temp\IO.dat;

# -t4: 4 threads
# -o32 outstanding IO
# -b512k block size
# -w0 wirte percentage
# -d45 duration 45 seconds
# -L  Measures latency statistics.
# -D: Captures IOPS statistics, such as standard deviation, in intervals of milliseconds (per-thread, per-target).
# -Suw: Disables software and hardware write caching (equivalent to -Sh).

```
