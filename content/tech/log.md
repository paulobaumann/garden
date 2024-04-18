---
title: log
date: 2023-10-20
tags: 
- development
---

On Linux, to follow the content being written to a file, we use `tail -F`.

The equivalent command in Windows (to be executed in PowerShell) is `Gc`. Say
we want to follow what is being written in the file `foo.log` from the directory
defined by the environmental variable `TEMP`:

    cd $Env:TEMP
    Gc foo.log -Wait
