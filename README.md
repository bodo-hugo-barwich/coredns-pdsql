---
title: "pdsql"
description: "*pdsql* use powerdns generic sql as backend."
weight: 10
tags: [ "plugin" , "pdsql" ]
categories: [ "plugin", "external" ]
date: "2017-12-09T10:26:00+08:00"
repo: "https://github.com/wenerme/coredns-pdsql"
home: "https://github.com/wenerme/coredns-pdsql/blob/master/README.md"
---

# pdsql

*pdsql* use PowerDNS [generic sql](https://github.com/PowerDNS/pdns/tree/master/pdns/backends/gsql) as backend.

Use [gorm.io/gorm](https://gorm.io) to handle database, support many database as gorm dose.

## Syntax

~~~ txt
pdsql <dialect> <arg> {
    # enable debug mode
    debug [db]
    # create table for test
    auto-migrate
}
~~~

## Install Driver

pdsql need db driver for dialect, current gorm do not support auto install driver, the supported driver is bundled with
this plugin.

- sqlite,sqlite3
- mysql
- postgres

## Examples

Start a server on the 1053 port, use test.db as backend.

~~~ corefile
test.:1053 {
    pdsql sqlite3 ./test.db {
        debug db
        auto-migrate
    }
}
~~~

Prepare data for test.

~~~ bash
# Insert records for wener.test
sqlite3 ./test.db 'insert into records(name,type,content,ttl,disabled)values("wener.test","A","192.168.1.1",3600,0)'
sqlite3 ./test.db 'insert into records(name,type,content,ttl,disabled)values("wener.test","TXT","TXT Here",3600,0)'
~~~

When queried for "wener.test. A", CoreDNS will respond with:

~~~ txt
;; QUESTION SECTION:
;wener.test.			IN	A

;; ANSWER SECTION:
wener.test.		3600	IN	A	192.168.1.1
~~~

When queried for "wener.test. ANY", CoreDNS will respond with:

~~~ txt
;; QUESTION SECTION:
;wener.test.			IN	ANY

;; ANSWER SECTION:
wener.test.		3600	IN	A	192.168.1.1
wener.test.		3600	IN	TXT	"TXT Here"
~~~

### Wildcard

~~~ bash
# domain id 1
sqlite3 ./test.db 'insert into domains(name,type)values("example.test","NATIVE")'
sqlite3 ./test.db 'insert into records(domain_id,name,type,content,ttl,disabled)values(1,"*.example.test","A","192.168.1.1",3600,0)'
~~~

When queried for "first.example.test. A", CoreDNS will respond with:

~~~ txt
;; QUESTION SECTION:
;first.example.test.		IN	A

;; ANSWER SECTION:
first.example.test.	3600	IN	A	192.168.1.1
~~~

## TODO

* [x] support wildcard record

