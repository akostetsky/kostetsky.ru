<!--
.. title: Как узнать по именами файла на каких машинах расположены блоки данных.
.. slug: how-to-track-which-data-block-is-in-which-datanode-in-hadoop-for-known-file
.. date: 2018-02-03 23:17:06 UTC+03:00
.. tags: hadoop, hdfs
.. category: hadoop
.. link: 
.. description: 
.. type: text
-->

```sh
$ sudo -u hdfs
# hdfs fsck <hdfs_file_path> -locations -blocks -files
```

