---
title: kmail
date: 2023-12-19
tags: 
- linux
---

kmail stores content using postgresql. The version of the database is 12.
However, after upgrading openSUSE, the postgresql was upgraded to 15.5. Now,
when I try to launch kmail, it fails and this is printed in the system log:

    Dec 18 23:47:49 linux-9cho akonadiserver[7165]: org.kde.pim.akonadiserver: Cluster PG_VERSION is 12 , PostgreSQL server is version  15 , will attempt to upgrade the cluster
    Dec 18 23:47:49 linux-9cho akonadiserver[7165]: org.kde.pim.akonadiserver: Postgres db cluster upgrade failed, Akonadi will fail to start. Sorry.
    Dec 18 23:47:49 linux-9cho akonadiserver[7165]: org.kde.pim.akonadiserver: Database process exited unexpectedly during initial connection!
    Dec 18 23:47:49 linux-9cho akonadiserver[7165]: org.kde.pim.akonadiserver: executable: "/usr/bin/pg_ctl"
    Dec 18 23:47:49 linux-9cho akonadiserver[7165]: org.kde.pim.akonadiserver: arguments: ("start", "-w", "--timeout=10", "--pgdata=/home/paulo/.local/share/akonadi/db_data", "-o \"-k/tmp/akonadi-paulo.hash\" -h ''")
    Dec 18 23:47:49 linux-9cho akonadiserver[7165]: org.kde.pim.akonadiserver: stdout: "esperando o servidor iniciar....2023-12-18 23:47:49.039 -03   [7172]FATAL:  database files are incompatible with server\n2023-12-18 23:47:49.039 -03   [7172]DETAIL:  The data directory was initialized by PostgreSQL version 12, which is not compatible with this version 15.5.\nparou de esperar\n"
    Dec 18 23:47:49 linux-9cho akonadiserver[7165]: org.kde.pim.akonadiserver: stderr: "pg_ctl: n\xC3\xA3o pode iniciar o servidor\nExamine o arquivo de log.\n"
    Dec 18 23:47:49 linux-9cho akonadiserver[7165]: org.kde.pim.akonadiserver: exit code: 1
    Dec 18 23:47:49 linux-9cho akonadiserver[7165]: org.kde.pim.akonadiserver: process error: "Unknown error"


The bin folder of those versions are:

    /usr/lib/postgresql12/bin
    /usr/lib/postgresql15/bin

To upgrade the DB:

    cd ~/.local/share/akonadi
    initdb db_data
    pg_upgrade -b /usr/lib/postgresql12/bin -d db_data.old -D db_data

Result: 

    lc_collate values for database "template1" do not match:  old "en_US.UTF-8", new "pt_BR.UTF-8" Failure, exiting

Second try: 

    rm -r db_data
    initdb --locale=en_US db_data
    pg_upgrade -b /usr/lib/postgresql12/bin -d db_data.old -D db_data
    encodings for database "template1" do not match:  old "UTF8", new "LATIN1" Failure, exiting

3rd try: 

    rm -r db_data
    initdb --locale=en_US.UTF-8 db_data
    pg_upgrade -b /usr/lib/postgresql12/bin -d db_data.old -D db_data

