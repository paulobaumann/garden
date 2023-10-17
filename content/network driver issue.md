---
title: Network driver issue
date: 2016-07-22
---

The processes `kworker` and `rsyslogd` were consuming a lot of CPU and flooding the files
`/var/log/syslog` and `/var/log/kern.log`. These files were growing at a rate of 1 GB
per minute, repeating messages such as:

```
pcieport 0000:00:1c.6: PCIe Bus Error: severity=Corrected, type=Physical Layer, id=00e6(Receiver ID)
pcieport 0000:00:1c.6:   device [8086:a116] error status/mask=00000001/00002000
pcieport 0000:00:1c.6:    [ 0] Receiver Error        
pcieport 0000:00:1c.6: AER: Corrected error received: id=00e6
pcieport 0000:00:1c.6: can't find device of ID00e6
```

The same messages also appear after running
[`dmesg`](http://askubuntu.com/a/421916/275635). Another tool for analyzing
kernel tasks is [[perf]].

To check for computer specifications, run:

```bash
inxi -Fxz
``` 

And this is part of the output:

```
System:    Host: xxxxx Kernel: 3.19.0-65-generic x86_64 (64 bit, gcc: 4.8.4) 
           Desktop: Gnome  (Gtk 2.24.23) Distro: Ubuntu 14.04 trusty
...
Network:   Card-1: Realtek RTL8723BE PCIe Wireless Network Adapter driver:
           rtl8723be port: d000 bus-ID: 02:00.0
           IF: wlan0 state: down mac: <filter>
           Card-2: Realtek RTL8111/8168/8411 PCI Express Gigabit Ethernet Controller 
           driver: r8169 ver: 2.3LK-NAPI port: e000 bus-ID: 01:00.0
           IF: eth0 state: up speed: 1000 Mbps duplex: full mac: <filter>
...
```

This same problem was also described in a [post in Ubuntu
Forums](https://ubuntuforums.org/showthread.php?t=2313100).

From the beginning of the error message (`pcieport 0000:00:1c.6`), I discovered
the culprit device [this way](http://unix.stackexchange.com/a/220651/146294):

```bash
lspci -v | grep -A 6 1c.6
lspci -s 2:0
```

Output:

```
00:1c.6 PCI bridge: Intel Corporation Sunrise Point-H PCI Express Root Port #7
	(rev f1) (prog-if 00 [Normal decode])
	Flags: bus master, fast devsel, latency 0
	Bus: primary=00, secondary=02, subordinate=02, sec-latency=0
	I/O behind bridge: 0000d000-0000dfff
	Memory behind bridge: df000000-df0fffff
	Capabilities: <access denied>
	Kernel driver in use: pcieport

02:00.0 Network controller: Realtek Semiconductor Co., Ltd. RTL8723BE PCIe
Wireless Network Adapter
```

The source code of the device is located at:
`/var/lib/dkms/rtk-btusb-dw1801-8723be/3/build`


## Cause

It is likely a problem in the build. Apparently some source files contain bugs.
This is the content of `/var/lib/dkms/rtk-btusb-dw1801-8723be/3/build/make.log`:

```
DKMS make.log for rtk-btusb-dw1801-8723be-3 for kernel 4.8.0-41-generic (x86_64)
qui mar 23 10:30:11 BRT 2017
make: Entering directory '/var/lib/dkms/rtk-btusb-dw1801-8723be/3/build'
make -C /lib/modules/4.8.0-41-generic/build M=/var/lib/dkms/rtk-btusb-dw1801-8723be/3/build modules
make[1]: Entering directory '/usr/src/linux-headers-4.8.0-41-generic'
  CC [M]  /var/lib/dkms/rtk-btusb-dw1801-8723be/3/build/rtk_coex.o
/var/lib/dkms/rtk-btusb-dw1801-8723be/3/build/rtk_coex.c: In function ‘update_profile_connection’:
/var/lib/dkms/rtk-btusb-dw1801-8723be/3/build/rtk_coex.c:417:5: warning: this ‘if’ clause does not guard... [-Wmisleading-indentation]
     if (profile_index < 0)
     ^~
/var/lib/dkms/rtk-btusb-dw1801-8723be/3/build/rtk_coex.c:420:2: note: ...this statement, but the latter is misleadingly indented as if it is guarded by the ‘if’
  if(is_add) {
  ^~
/var/lib/dkms/rtk-btusb-dw1801-8723be/3/build/rtk_coex.c: In function ‘udpsocket_send’:
/var/lib/dkms/rtk-btusb-dw1801-8723be/3/build/rtk_coex.c:719:17: error: too many arguments to function ‘sock_sendmsg’
         error = sock_sendmsg(usb_coex_info.udpsock, &udpmsg, msg_size);
                 ^~~~~~~~~~~~
In file included from ./include/linux/skbuff.h:29:0,
                 from /var/lib/dkms/rtk-btusb-dw1801-8723be/3/build/rtk_coex.c:8:
./include/linux/net.h:221:5: note: declared here
 int sock_sendmsg(struct socket *sock, struct msghdr *msg);
     ^~~~~~~~~~~~
scripts/Makefile.build:289: recipe for target '/var/lib/dkms/rtk-btusb-dw1801-8723be/3/build/rtk_coex.o' failed
make[2]: *** [/var/lib/dkms/rtk-btusb-dw1801-8723be/3/build/rtk_coex.o] Error 1
Makefile:1491: recipe for target '_module_/var/lib/dkms/rtk-btusb-dw1801-8723be/3/build' failed
make[1]: *** [_module_/var/lib/dkms/rtk-btusb-dw1801-8723be/3/build] Error 2
make[1]: Leaving directory '/usr/src/linux-headers-4.8.0-41-generic'
Makefile:11: recipe for target 'all' failed
make: *** [all] Error 2
make: Leaving directory '/var/lib/dkms/rtk-btusb-dw1801-8723be/3/build'
```


## Attempt #1

I've tried getting a new `rtl8723befw` from Ubuntu repo, as suggested on [this
post from Ask Ubuntu](http://askubuntu.com/a/673826/275635):

```bash 
cd /lib/firmware/rtlwifi/
sudo mv rtl8723befw.bin rtl8723befw.bin.bkp
sudo wget https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1454843/+attachment/4438588/+files/rtl8723befw.bin
echo "options rtl8723be fwlps=0" | sudo tee /etc/modprobe.d/rtl8723be.conf
reboot
``` 

It didn't work. I needed to undo the modifications.


## Attempt #2

I've tried changing some of the options of the firmware `rtl8723be`, as
suggested in a [post in
Dedoimedo](http://www.dedoimedo.com/computers/ubuntu-trusty-realtek.html):

```bash
modinfo rtl8723be 
echo "options rtl8723be fwlps=N ips=N" | sudo tee /etc/modprobe.d/rtl8723be.conf
reboot
```

It didn't work. I needed to undo the modifications.
I've even tried an alternative option, but without success as well:

```
echo "options rtl8723be fwlps=0 ips=0" | sudo tee /etc/modprobe.d/rtl8723be.conf
```


## Attempt #3

I've installed the firmware from [hanipouspilot
PPA](https://launchpad.net/~hanipouspilot). This approach followed instructions
taken from different posts in Ask Ubuntu (
([1](http://askubuntu.com/a/775483/275635), 
[2](http://askubuntu.com/a/629690/275635), 
[3](http://askubuntu.com/a/698398/275635)).

```bash
sudo add-apt-repository ppa:hanipouspilot/rtlwifi
sudo apt-get update
sudo apt-get install rtlwifi-new-dkms
sudo apt-get install linux-firmware
echo "options rtl8723be fwlps=N ips=N swenc=Y msi=1" | sudo tee /etc/modprobe.d/rtl8723be.conf
reboot
```

It worked for a few minutes, but then the logs started happening again. Other
combinations of parameters for the `rtl8723be.conf` file did not even work to
connect to the internet via Wi-Fi.


## Attempt #4

Disable PCI AER (Advanced Error Reporting), as suggested in a [post in this bug
report](https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1521173):

 1. edit `/etc/default/grub` and and add pci=noaer to the line starting with
    `GRUB_CMDLINE_LINUX_DEFAULT`. It will look like this:
    `GRUB_CMDLINE_LINUX_DEFAULT="quiet splash pci=noaer"`
 2. `sudo update-grub`
 3. reboot

The messages stopped flooding the log, but the error persists.


## Attempt #5

I replaced the driver with one provided by
[lwfinger](https://github.com/lwfinger/rtlwifi new). When following his
tutorial, the error occurred in the `sudo make install` step:

```
make -C /lib/modules/4.14.13-1-default/build M=/home/paulo/github/rtlwifi_new modules
make[1]: *** /lib/modules/4.14.13-1-default/build: Arquivo ou diretório não encontrado.  Pare.
make: *** [Makefile:58: all] Error 2
```

the problem was the missing link `/lib/modules/4.14.13-1-default/build`. This
was fixed following the suggestion given in this [post](https://forums.opensuse.org/showthread.php/491320-kernel-header-install?p=2592043#post2592043).

It wasn't enough to fix the problem. It continued to persist even after
reactivating the module. According to a [post](https://askubuntu.com/a/775483),
I ran:

```bash
echo "options rtl8723be fwlps=0 ips=0 ant_sel=2" | sudo tee /etc/modprobe.d/rtl8723be.conf
reboot
```

*This worked!* One can notice that in `dmesg` there is no longer any
notification like `pcieport 0000:00:1c.6: pcie bus error`, but rather `pcieport
0000:00:1c.6: signaling pme with irq 123`.

In summary, the final solution was:

```bash
git clone https://github.com/lwfinger/rtlwifi_new.git
cd rtlwifi_new
sudo make install
echo "options rtl8723be fwlps=0 ips=0 ant_sel=2" | sudo tee /etc/modprobe.d/rtl8723be.conf
sudo modprobe -r rtl8723be 
sudo modprobe rtl8723be
reboot  # perhaps
```


## Problem after reinstalling openSUSE

After reinstalling openSUSE, I was able to log into the system once. I saw the
error, but I disabled it via `modprobe -r rtl8723be`. However, after restarting
the computer, it no longer reached the login screen, as it kept displaying an
error message on the screen forever.

The solution was to boot into recovery mode with the pen drive I used to install
openSUSE. In its terminal, I followed these steps:

1. log in as `root`
2. check partitions: `fdisk -l`
3. mount the system partition: `mount /dev/sda3 /mount`
4. open module blacklist: `vi /etc/modprobe.d/50-blacklist.conf`
5. insert the defective module at the end: `blacklist rtl8723be`

After that, I restarted the PC and it booted correctly.


## Workaround

It is possible to disable the driver, thus ceasing kworker and rsyslogd
activity, but leaving no Wi-Fi:

```bash
sudo modprobe -rv rtl8723be
```



