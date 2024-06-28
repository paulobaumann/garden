---
title: Termux
date: 2024-05-22
tags: 
- android
- linux
---

Termux is a terminal emulator application for Android OS extendible by variety of packages.

## Download

The version available on Google Play is [old and has
vulnerabilities](https://github.com/termux/termux-app?tab=readme-ov-file#installation).
Prefer [F-Droid](https://f-droid.org/) app store. To get the F-Droid,
dowload the APK file to your PC and copy it to your phone. Tip: use Dropbox.

## Setup

On Termux terminal:

    pkg update
    pkg install openssh

Set a password for Termux:

    passwd

Start the SSH server in Termux:

    sshd

Find your Phone's IP address:

    ifconfig

Find your username:

    whoami

SSH into your phone from your computer:

    ssh -p 8022 username@phone_ip
