---
layout: post
title: "Transfer files between nix based machines"
date: 2019-02-16 00:00:00 +0530
author: Mallikarjun
categories:
permalink: transfer-files-between-nix-based-machines
tags: scp nc rsync files linux
---

## Overview

1. [Transfer files using netcat](#transfer-files-using-netcat)
2. [Transfer files using python simplehttpserver](#transfer-files-using-python-simplehttpserver)
3. [References](#references)

## Transfer files using netcat
Consider you have a file `/tmp/mini.iso`. You want to transfer from your machine to `10.0.0.11` 

Start netcat server on destination machine listening on `5555` output to destination file

```bash
[~] ➜ nc -l -p 5555 > /tmp/mini.iso
```

Send file from source machine to the port on which destination machine is listening on

``` bash
[~] ➜ cat /tmp/mini.iso | pv | nc 10.0.0.11 5555
```
OR
```bash
[~] ➜ nc 10.0.0.11 5555 < /tmp/mini.iso
```

**&raquo;** `nc` -- arbitrary TCP and UDP connections and listens

**&raquo;** `pv` -- monitor the progress of data through a pipe

**&raquo;** `cat` -- concatenate files and print on the standard output

**&raquo;** `-l` -- listen on specified port

**&raquo;** `-p` -- port to be listened on


## Transfer files using python simplehttpserver

Consider you have a file `/tmp/mini.iso`. You want to transfer from your machine to `10.0.0.11` 

Start python simple httpserver on the source machine listening on `5555` from the folder in which your file is placed

```bash
[~] ➜ ls
photo-1525249181835

[~] ➜ python -m SimpleHTTPServer 5555
```

On destination machine, open the browser and visit link `http://10.0.0.11:5555/`. You should see following information. Then you can download from the same

```
Directory listing for /
photo-1525249181835
```

## References
**&raquo;** [Linux Man pages](https://www.kernel.org/doc/man-pages/)

**&raquo;** [Netcat Man pages](http://man7.org/linux/man-pages/man1/ncat.1.html)

**&raquo;** [PV Man pages](http://man7.org/linux/man-pages/man1/pv.1.html)
