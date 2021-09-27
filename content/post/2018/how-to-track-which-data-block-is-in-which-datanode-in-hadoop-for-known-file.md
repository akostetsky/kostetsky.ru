---
title: Как узнать по имени файла на каких машинах расположены блоки данных.
date: 2018-02-03T12:45:00+03:00
tags:
  - hdfs
  - hadoop
topics:
  - hadoop
---
```sh
$ sudo -u hdfs
# hdfs fsck <hdfs_file_path> -locations -blocks -files
```
