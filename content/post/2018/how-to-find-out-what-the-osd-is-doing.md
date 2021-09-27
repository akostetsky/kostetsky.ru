---
title: Как узнать, чем занята OSD
date: 2018-02-03T15:34:00+03:00
tags:
  - ceph
  - osd
topics:
  - ceph
---
Что бы узнать, чем занята OSD, нужно использовать следующий синтаксис:
```sh
ceph daemon osd.{id} ops
```
, где {id} - id osd.