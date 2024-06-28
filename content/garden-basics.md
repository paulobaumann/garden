---
title: Garden basics
---

This site is written using [GitHub Flavored Markdown](https://github.github.com/gfm/) and
hosted in [GitHub Pages](https://paulobaumann.github.io/garden/).

The original repository from [jackyzha](https://quartz.jzhao.xyz/) is configured
as remote repo named `quartz`. But since I only intend to fetch from and not
push to it, I have removed the push URL by running:

    git remote set-url --push quartz ""

I get quartz latest updates running (assuming `v4` is the main branch on the
quartz repo):

    git fetch quartz
    git rebase quartz/v4

My changes are commited to branch `main`. I can see it locally by running:

    npx quartz build --serve


## On Android

### Using Termux

BLUF: Failed.

Installed [[Termux]].
Installed Quartz package, but it failed: 

> Error: Cannot find module '../lightningcss.android-arm64.node'

This package does not support Android platform. I've tried
[lightningcss-linux-arm64-gnu](https://www.npmjs.com/package/lightningcss-linux-arm64-gnu),
but didn't installed either. 

It does not seem to have a way to bypass this packge on Quartz. So, building
Quartz on Android is not possible through Termux.

### Using UserLAnd

BLUF: Worked!

Installed [[UserLAnd]]. Then Quartz:

    apk add nodejs npm vim git
    git clone https://github.com/jackyzha0/quartz.git
    cd quartz
    npm i 
    npx quartz create

But accessing UserLAnd's storage directly via Android apps can be challenging
because UserLAnd uses its own isolated file system environment. However, it is
possible to use Termux as a Bridge. Accessing shared storage in Termux:

    termux-setup-storage
    cd /storage/emulated/0

In UserLAnd, navigate to the Termux shared storage path and copy files:

    cp /path/to/file /storage/emulated/0/Download/

However, installing `quartz` did not work because it is not compatible with the
system architecture.
