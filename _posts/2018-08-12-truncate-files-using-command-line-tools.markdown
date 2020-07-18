---
layout: post
title: Truncate files using unix command tools
date: 2018-08-12 00:00:00 +0530
author: Mallikarjun
categories:
permalink: truncate-files-using-command-line-tools
tags: file truncate command tool command-line
---

## Overview

1. [Empty the contents of file](#empty-the-contents-of-file)
2. [Truncate tail end of the file contents](#truncate-tail-end-of-the-file-contents)
3. [Truncate top end of the file contents](#truncate-top-end-of-the-file-contents)
4. [Truncate arbitrary part of the file contents](#truncate-arbitrary-part-of-the-file-contents)
5. [Truncate the file contents by number of lines](#truncate-the-file-contents-by-number-of-lines)
6. [References](#references)

## Empty the contents of file
	
This command empties the file using redirection operator. From my experiments using `time` command, it is found to be faster than methods shown below.

``` bash
$ ls -l file.txt
-rw-r--r-- 1 none none 5019968 Mar 31 18:32 file.txt
$ > file.txt
$ ls -l file.txt
-rw-r--r-- 1 none none 0 Mar 31 20:50 file.txt
```


**&raquo;** redirection operator is a way of redirecting information from [standard streams](http://en.wikipedia.org/wiki/Standard_streams) to user defined locations

This command truncates the size of the file to `0` bytes. Also, you can fill up file with `NULL -- 00 hex` values of specific size greater than the actual file size, for example `> file.txt; truncate --size 1024 file.txt` is perfectly valid command.

``` bash
$ ls -l file.txt
-rw-r--r-- 1 none none 5019968 Mar 31 18:32 file.txt
$ truncate --size 0 file.txt
$ ls -l file.txt
-rw-r--r-- 1 none none 0 Mar 31 20:50 file.txt
```

**&raquo;** `truncate` : command to shrink or enlarge a file to a specified size.

**&raquo;** `--size` : argument specifies the size to which file should be shrunk or enlarged to.

This command uses [null device](http://en.wikipedia.org/wiki//dev/null) to write out empty data to `file.txt`. There is another way to do the same, `cp /dev/null file.txt`, which I think is slower than the above methods.

``` bash
$ ls -l file.txt
-rw-r--r-- 1 none none 5019968 Mar 31 18:32 file.txt
$ cat /dev/null > file.txt
$ ls -l file.txt
-rw-r--r-- 1 none none 0 Mar 31 20:50 file.txt
```

**&raquo;** `/dev/null` :  is a special device called [null device](http://en.wikipedia.org/wiki//dev/null) on Unix-like operating systems which provides no data when you read from it nor does it store any data when you write to it.

## Truncate tail end of the file contents

When you truncate a file to a specific size, be sure that actual size of the file is greater than the specified size and `truncate` command always removes/adds from/to the tail end of the file. If file is to be enlarged, it fills up the rest of the space with unix null character (\x0).

``` bash
$ ls -l file.txt
-rw-r--r-- 1 none none 5019968 Mar 31 18:32 file.txt
$ truncate --size 1024 file.txt
$ ls -l file.txt
-rw-r--r-- 1 none none 1024 Mar 31 20:50 file.txt
```

If you want to trim off some arbitrary size based on the actual size of the file, say trim by half the size. Then you can use `awk` and `wc` to calculate the size to trim.

``` bash
$ ls -l file.txt
-rw-r--r-- 1 none none 5019968 Mar 31 18:32 file.txt
$ wc file.txt | truncate --size `awk -F " " '{printf "%d\n", ($3/2)}'` file.txt
$ ls -l file.txt
-rw-r--r-- 1 none none 2509984 Mar 31 20:57 file.txt
```

**&raquo;** `wc` : command to print lines, words and bytes count read from a stream

**&raquo;** `awk` : a handy yet powerful text processing language.

**&raquo;** `-F` or `--field-separator` : to specify input stream field separator, in this case it is spaces. 

## Truncate top end of the file contents

There isn't a easy way to truncate arbitrary part of the file, `split` does the trick but it requires twice the file size. `split` helps in splitting the file into 2 parts(or any number of parts using `--number=n` parameter) and `mv` to rename part which we want into the actual file. 

``` bash
$ ls -l file.txt
-rw-r--r-- 1 none none 133054207 Apr  6 10:15 file.txt
$ split --suffix-length=1 --number=2 file.txt output; mv outputa file.txt; rm outputb
$ ls -l file.txt 
-rw-r--r-- 1 none none 66527103 Apr  6 10:16 file.tx
```

**&raquo;** `split` : split the files in various ways, checkout `man` pages for more info. This takes a prefix at the end of the command, which is used as a prefix to the newly created split files, for example `output` is the prefix we have used above.

**&raquo;** `-a` or `--suffix-length` : is the length of the suffix used on the split files to distinguish between them. For example, we have outputa and outputb in the above example

**&raquo;** `-n` or `--number` : is the number of files to generate as split output.

**&raquo;** `mv` : rename files.

**&raquo;** `rm` : remove files.

**WARNING:** This command needs twice the disk space of the file you are truncating.

## Truncate arbitrary part of the file contents

This is very similar previous section except we have changed a few things like `--number=3` and the choice of the split file we are interested in. 

``` bash
$ ls -l file.txt
-rw-r--r-- 1 none none 133054207 Apr  6 10:15 file.txt
$ split --suffix-length=1 --number=3 file.txt output; mv outputb file.txt; rm outputa outputc
$ ls -l file.txt 
-rw-r--r-- 1 none none 44351402 Apr  6 10:16 file.tx
```

**WARNING:** This command needs twice the disk space of the file you are truncating.

## Truncate the file contents by number of lines

This again is very similar to previous section except that we use `--lines` argument of split. 

``` bash
$ wc file.txt 
994046  3574572 44351402 file.txt
$ split --suffix-length=1 --lines=100000 file.txt output; mv outputa file.txt; rm output*;
$ wc file.txt 
100000  359611 4460876 file.txt
```

**&raquo;** `--lines` : This is a `split` command argument by we can choose the number of lines each split file contains.

**WARNING:** This command needs twice the disk space of the file you are truncating.

You have to do a little more circus to retain last n lines. Because if you use `split`, last file isn't guaranteed to be n lines for obvious reasons and hard to determine the last split chunk file name. This method is least efficient of the previous ones because of the use of `cat` instead of `mv` as in earlier cases.

``` bash
$ wc file.txt
994046  3574572 44351402 file.txt
$ wc file.txt | split --suffix-length=1 --lines=`awk -F " " '{printf "%d\n", ($1-100000)}'` file.txt output; rm outputa; cat output* > file.txt
$ wc file.txt 
100000  359611 4460876 file.txt
```

## References
**&raquo;** [Linux Man pages](https://www.kernel.org/doc/man-pages/)
