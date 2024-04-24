---
title: backup
date: 2023-10-18
tags: 
- linux
---

Backups are a must, specially before a OS [[openSUSE upgrade | upgrade]]. 

To easily share the Linux configuration files (aka dotfiles) to other machines,
I've followed the suggestion presented by the Atlassian tutorial [how to store
dotfiles](https://www.atlassian.com/git/tutorials/dotfiles). Now I have my
`.vimrc` in a private github repo called
[cfg](https://github.com/paulobaumann/cfg). It worked like a charm!

To backup other files to a external HD, I use rsync command to mirror the
content of some folders from my PC into the external HD. 

    # Choose one of the following external HDs:
    #HD="Seagate\ Expansion\ Drive"
    HD="TOSHIBA\ EXT"
    rsync -vaXmh --stats --delete /home/paulo/Documents/ "/run/media/paulo/$HD/paulo/Documents/"

A special comment about the flag `--delete`:

- This mirrors the files from the source folder (my PC, which holds the files to
  be backed up) to the destination folder (external HD).
- Files in the destination folder that differ from the source will be DELETED.
- Do not use this flag to back up folders where you don't want to mirror (e.g.,
  folder `music` in my PC have only a share of the files while the external HD
  has the whole collection).

The remaining flags have this meaning:

    -v, --verbose
    -a, --archive           archive mode; equals -rlptgoD (no -H,-A,-X)
    -r, --recursive         recursive
    -l, --links             copy symlinks as symlinks
    -p, --perms             preserve permissions
    -t, --times             preserve modification times
    -g, --group             preserve group
    -o, --owner             preserve owner (super-user only)
    -D                      same as --devices --specials
    --devices               preserve device files (super-user only)
    --specials              preserve special files
    -X, --xattrs            preserve extended attributes
    -m, --prune-empty-dirs  prune empty directory chains from file-list
    -h, --human-readable    output numbers in a human-readable format
