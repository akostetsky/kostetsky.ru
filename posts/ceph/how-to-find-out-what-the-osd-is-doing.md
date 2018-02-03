<!--
.. title: Как узнать, чем занята OSD
.. slug: how-to-find-out-what-the-osd-is-doing
.. date: 2018-02-03 23:54:49 UTC+03:00
.. tags: ceph, osd
.. category: ceph
.. link: 
.. description: 
.. type: text
-->

Что бы узнать, чем занята OSD, нужно использовать следующий синтаксис:

```sh
ceph daemon osd.{id} ops
```
, где {id} - id osd.


