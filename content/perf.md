---
title: perf
date: 2023-10-15
tags: 
- linux
---

[`perf`](http://askubuntu.com/a/422151/275635) is a tool for analyzing kernel
tasks. To install it using apt package manager:

```
sudo apt-get install linux-tools-common linux-tools-3.19.0-65-generic
sudo perf record -g -a sleep 10
sudo perf report
``` 
