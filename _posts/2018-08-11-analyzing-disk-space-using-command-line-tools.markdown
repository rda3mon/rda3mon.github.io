---
layout: post
title:  "Analyzing disk space using command line tools"
date:   2018-08-11 00:00:00 +0530
author: Mallikarjun
categories:
permalink: analyzing-disk-space-using-command-line-tools
tags: du disk command tool command-line
---

## Overview
1. [Disk usage of all files in a particular path](#disk-usage-of-all-files-in-a-particular-path)
2. [Disk usage of all files in a particular path with a particular depth](#disk-usage-of-all-files-in-a-particular-path-with-a-particular-depth)
3. [Disk usage of all directories in a particular directory](#disk-usage-of-all-directories-in-a-particular-directory)
4. [Disk usage of all files in a particular path with a particular depth and exclude certain files](#disk-usage-of-all-files-in-a-particular-path-with-a-particular-depth-and-exclude-certain-files)
5. [Disk usage of all files in a particular path with sorted order](#disk-usage-of-all-files-in-a-particular-path-with-sorted-order)
6. [Disk usage of top 10 files in a particular path](#disk-usage-of-top-10-files-in-a-particular-path)
7. [Disk usage of all files between certain file size range](#disk-usage-of-all-files-between-certain-file-size-range)
8. [References](#references)

## Disk usage of all files in a particular path
This command scans the `/var/log/*` directory recursively for all the paths.

``` bash
$ du /var/log/*
1364	/var/log/samba/cores/smbd
4	/var/log/samba/cores/nmbd
444	/var/log/upstart
292	/var/log/wtmp
92	/var/log/Xorg.0.log
28	/var/log/Xorg.20.log
```


**&raquo;** `du` is a command line utility to check disk space usage.

## Disk usage of all files in a particular path with a particular depth
This command scans the `/var/log/*` directory recursively with a maximum directory depth of 1. By `--max-depth=1` it means to say that show space usage of all the paths that can recursively go up to one more level under `/var/log/`, which include `/var/log/*` and `/var/log/*/`(on further levels - directories only), and not `/var/log/*/*/`

``` bash
$ du /var/log/* -h --max-depth=1
1.4M	/var/log/samba/cores
1.5M	/var/log/samba
444K	/var/log/upstart
296K	/var/log/wtmp
92K	/var/log/Xorg.0.log
28K	/var/log/Xorg.20.log
```

**&raquo;** `--max-depth` is the depth up to which the command should recurse. 0 being minimum (only current directory)

**&raquo;** `--h` to make the reading human readable

## Disk usage of all directories in a particular directory
This commands scans `/var/log/` for all the directories rather than files. The difference between earlier commands is that non-use of `*` displays disk space consumption of only directory paths with displaying files under them.

``` bash
$ du /var/log/ -h --max-depth=1
1.5M	/var/log/samba
444K	/var/log/upstart
4.0K	/var/log/news
4.0K	/var/log/tomcat6
14M	/var/log/
```

## Disk usage of all files in a particular path with a particular depth and exclude certain files

This command scans the `/var/log/` directory recursively with a maximum directory depth of 1 and excluding all files ending with `.log` extension. This command not excludes only displaying the files matching pattern, but also removes space consumption when displaying those matching files parent directory

``` bash
$ du /var/log/* --max-depth=1 --exclude=*.log
1372	/var/log/samba/cores
1436	/var/log/samba
444	/var/log/upstart
296	/var/log/wtmp
```

**&raquo;** `--exclude` exclude all files that match certain pattern

## Disk usage of all files in a particular path with sorted order

This command scans the `/var/log/` directory recursively with a maximum directory depth of 1 level and sorts the first column(file sizes) in numeric order.

``` bash
$ du /var/log/* --max-depth=1 | sort -k1 -n
28	/var/log/Xorg.20.log
92	/var/log/Xorg.0.log
296	/var/log/wtmp
444	/var/log/upstart
1372	/var/log/samba/cores
1436	/var/log/samba
```

**&raquo;** `sort` is a command line utility to sort output

**&raquo;** `-k` is the column number -- we need only first column which has disk space

**&raquo;** `-n` to sort in numeric format

`Caution`: Do not use `-h`, will lead to improper sorting.

## Disk usage of top 10 files in a particular path

This command scans `/var/log/` folder recursively and sorts them in decreasing order of file size displaying only top 10 results.

``` bash
$ du /var/log/* --max-depth=0 | sort -k1 -n | tail -10 | tac
1436	/var/log/samba
444	/var/log/upstart
296	/var/log/wtmp
92	/var/log/Xorg.0.log
28	/var/log/Xorg.20.log
```

**&raquo;** `tail` is a command which shows only the last n lines of output

**&raquo;** `tac` does reverse of `cat` command -- lines in reverse order

## Disk usage of all files between certain file size range

This command scans `/var/log/` folder printing files with size between 30KB and 1400KB.

``` bash
$ du /var/log/* --max-depth=0 | sort -k1 -n | awk '$1 > 30 && $1 < 1400'
444	/var/log/upstart
296	/var/log/wtmp
92	/var/log/Xorg.0.log
```

**&raquo;** `awk` is a powerful data extraction and reporting tool

## References
**&raquo;** [Linux Man pages](https://www.kernel.org/doc/man-pages/)
