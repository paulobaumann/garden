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
