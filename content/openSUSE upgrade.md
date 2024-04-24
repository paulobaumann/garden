---
title: openSUSE upgrade
date: 2023-10-18
tags: 
- linux
---

I had some issues upgrading openSUSE 15.2. In one of those attempts, I needed
to roolback the system to a snapshot of the system prior to the upgrade. These
were the steps:

1. Boot to the desired snapshot (use F12 during boot)
2. Check if the system status is OK
3. At the terminal, run: `sudo snapper rollback`

Some tips that helped during the upgrade from openSUSE 15.2 were taken from [a post
in Linux Kamarada](https://linuxkamarada.com/pt/2021/12/23/linux-kamarada-e-opensuse-leap-como-atualizar-da-versao-152-para-a-153/):

- remove virtualbox-kmp-default package (indeed, this package seemed to be the
  culprit for the upgrade failing)
- run `zypper dup` first time with `--download-only` flag
- run it a second time, but on a [virtual
  terminal](https://tldp.org/LDP/GNU-Linux-Tools-Summary/html/virtual-terminals.html)
  (Ctrl+Alt+F1) logged as root and with
  [runlevel](https://en.opensuse.org/SDB:Switch_runlevel) 3: `init 3`
