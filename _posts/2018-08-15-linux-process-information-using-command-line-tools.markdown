---
layout: post
title: "Linux process status using command line tools"
date: 2018-08-15 00:00:00 +0530
author: Mallikarjun
categories:
permalink: linux-process-information-using-command-line-tools
tags: ps top process command-line process-status
---

## Overview

1. [List all processes in system](#list-all-processes-in-system)
2. [List processes by users](#list-processes-by-users)
3. [List process tree](#list-process-tree)
4. [Process status Wrap output](#process-status-wrap-output)
5. [Show process thread information](#show-process-thread-information)
6. [Search processes by command](#search-processes-by-command)
7. [Search process by pid and output memory and cpu](#search-process-by-pid-and-output-memory-cpu-and-threads)
8. [Search process by pid and output memory and cpu in repeated interval](#search-process-by-pid-and-output-memory-and-cpu-in-repeated-interval)
9. [Sort processes by memory, cpu](#sort-processes-by-memory-cpu)
10. [Processess view using Top command](#processess-view-using-top-command)
11. [References](#references)

## List all processes in system
This command lists all processes currently running in system

``` bash
➜ ps -eF
UID        PID  PPID  C    SZ   RSS PSR STIME TTY          TIME CMD
root         1     0  0 29957  5916   6 Aug14 ?        00:00:03 /sbin/init splash
root         2     0  0     0     0   5 Aug14 ?        00:00:00 [kthreadd]
root         3     2  0     0     0   0 Aug14 ?        00:00:01 [ksoftirqd/0]
root         5     2  0     0     0   0 Aug14 ?        00:00:00 [kworker/0:0H]
root         7     2  0     0     0   0 Aug14 ?        00:00:38 [rcu_sched]
root         8     2  0     0     0   0 Aug14 ?        00:00:00 [rcu_bh]
root         9     2  0     0     0   0 Aug14 ?        00:00:02 [migration/0]
root        10     2  0     0     0   0 Aug14 ?        00:00:00 [watchdog/0]
root        11     2  0     0     0   1 Aug14 ?        00:00:00 [watchdog/1]
```

**&raquo;** `ps` is a command line utility to report a snapshot of current processes.

**&raquo;** `-e` selects all processes

## List processes by users 
This command lists the processes filtered by set of user

``` bash
➜ ps -F -u root,syslog
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 Aug14 ?        00:00:03 /sbin/init splash
root         2     0  0 Aug14 ?        00:00:00 [kthreadd]
root         3     2  0 Aug14 ?        00:00:00 [ksoftirqd/0]
root         5     2  0 Aug14 ?        00:00:00 [kworker/0:0H]
root         7     2  0 Aug14 ?        00:00:24 [rcu_sched]
root         8     2  0 Aug14 ?        00:00:00 [rcu_bh]
root         9     2  0 Aug14 ?        00:00:01 [migration/0]
syslog     921     1  0 Aug14 ?        00:00:00 /usr/sbin/rsyslogd -n
```

**&raquo;** `-f` to display more information about the processes

**&raquo;** `-u` to display the processes of set of users

## List process tree
Lists all processes in tree format

```bash
➜ ps -ef --forest
UID        PID  PPID  C STIME TTY          TIME CMD
root         2     0  0 Nov08 ?        00:00:00 [kthreadd]
root         3     2  0 Nov08 ?        00:00:06  \_ [ksoftirqd/0]
root         5     2  0 Nov08 ?        00:00:00  \_ [kworker/0:0H]
root         7     2  0 Nov08 ?        00:00:57  \_ [rcu_sched]
root     27466     1  0 11:02 ?        00:00:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
www-data 27467 27466  0 11:02 ?        00:00:00  \_ nginx: worker process
www-data 27468 27466  0 11:02 ?        00:00:00  \_ nginx: worker process
www-data 27469 27466  0 11:02 ?        00:00:00  \_ nginx: worker process
www-data 27474 27466  0 11:02 ?        00:00:00  \_ nginx: worker process

```

Lists single process in tree format.
```bash
ps --forest $(ps -e --no-header -o pid,ppid|awk -vp=<PID> 'function r(s){print s;s=a[s];while(s){sub(",","",s);t=s;sub(",.*","",t);sub("[0-9]+","",s);r(t)}}{a[$2]=a[$2]","$1}END{r(p)}')
  PID TTY      STAT   TIME COMMAND
27466 ?        Ss     0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
27467 ?        S      0:00  \_ nginx: worker process
27468 ?        S      0:00  \_ nginx: worker process
27469 ?        S      0:00  \_ nginx: worker process
27470 ?        S      0:00  \_ nginx: worker process
27471 ?        S      0:00  \_ nginx: worker process
27472 ?        S      0:00  \_ nginx: worker process
27473 ?        S      0:00  \_ nginx: worker process
27474 ?        S      0:00  \_ nginx: worker process
```

```bash
➜ pstree -p 27466
nginx(27466)─┬─nginx(27467)
             ├─nginx(27468)
             ├─nginx(27469)
             ├─nginx(27470)
             ├─nginx(27473)
             └─nginx(27474)
```

**&raquo;** `--forest` display the output in tree format

**&raquo;** `--no-header` to hide headers from ps output

**&raquo;** `-o args` arguments to be displayed in ps output. Refer man pages for details

## Process status Wrap output
This does not truncate the process status output full command

```bash
➜ ps -ef ww
nobody    6577  1009  0 Nov08 ?        S      0:01 /usr/sbin/dnsmasq --no-resolv --keep-in-foreground --no-hosts --bind-interfaces --pid-file=/var/run/NetworkManager/dnsmasq.pid --listen-address=127.0.1.1 --cache-size=0 --conf-file=/dev/null --proxy-dnssec --enable-dbus=org.freedesktop.NetworkManager.dnsmasq --conf-dir=/etc/NetworkManager/dnsmasq.d
```

**&raquo;** `ww` use this to set unlimited width. Or set `-w width` to set limited width

## Show process thread information
This shows threads as processes with entry for each thread. NLWP - number of threads, LWP - light weight process (thread) ID

```bash
ps -f -p 10064 H -L
UID        PID  PPID   LWP  C NLWP STIME TTY      STAT   TIME CMD
nobody 10064  9921 10064  0   13 Nov09 ?        Sl     3:00 /opt/google/chrome/chrome
nobody 10064  9921 10069  0   13 Nov09 ?        Sl     0:00 /opt/google/chrome/chrome
nobody 10064  9921 10071  0   13 Nov09 ?        Sl     0:00 /opt/google/chrome/chrome
nobody 10064  9921 10072  0   13 Nov09 ?        Sl     0:18 /opt/google/chrome/chrome
nobody 10064  9921 10075  0   13 Nov09 ?        Sl     0:00 /opt/google/chrome/chrome
nobody 10064  9921 10076  0   13 Nov09 ?        Sl     0:00 /opt/google/chrome/chrome
nobody 10064  9921 10079  0   13 Nov09 ?        Sl     0:00 /opt/google/chrome/chrome
nobody 10064  9921 10085  0   13 Nov09 ?        Sl     0:00 /opt/google/chrome/chrome
nobody 10064  9921 10086  0   13 Nov09 ?        Sl     0:00 /opt/google/chrome/chrome
nobody 10064  9921 10087  0   13 Nov09 ?        Sl     0:00 /opt/google/chrome/chrome
nobody 10064  9921 10088  0   13 Nov09 ?        Sl     0:00 /opt/google/chrome/chrome
nobody 10064  9921 10089  0   13 Nov09 ?        SNl    0:00 /opt/google/chrome/chrome
nobody 10064  9921 27031  0   13 14:31 ?        Sl     0:00 /opt/google/chrome/chrome
```

**&raquo;** `H` Show threads as if they were processes.

**&raquo;** `-L` Show threads, possibly with LWP and NLWP columns.

Print thread count for a particular process

```bash
➜ ps -o nlwp -p 10064
NLWP
  14
```

## Search processes by command

```bash
➜ ps -f -C chrome
UID        PID  PPID  C STIME TTY          TIME CMD
nobody  9904  4687  1 Nov09 ?        00:40:02 /opt/google/chrome/chrome
nobody  9916  9904  0 Nov09 ?        00:00:00 /opt/google/chrome/chrome --type=zygote
nobody  9921  9916  0 Nov09 ?        00:00:01 /opt/google/chrome/chrome --type=zygote
nobody 10064  9921  0 Nov09 ?        00:05:08 /opt/google/chrome/chrome
nobody 10112  9921  1 Nov09 ?        00:46:04 /opt/google/chrome/chrome
nobody 10359  9921  0 Nov09 ?        00:00:47 /opt/google/chrome/chrome
nobody 10364  9921  0 Nov09 ?        00:06:22 /opt/google/chrome/chrome
```

**&raquo;** `-C` filter by command

## Search process by pid and output memory cpu and threads

```bash
➜ ps -p 1113 -o pid,ppid,%mem,%cpu,nlwp,lwp
  PID  PPID %MEM %CPU NLWP   LWP
 1113  6054  1.2  0.6   47  1113

```

## Search process by pid and output memory and cpu in repeated interval

```bash
➜ watch -n 1 'ps -p 1113 -o pid,ppid,%mem,%cpu'
```

Output: 

```bash
Every 1.0s: ps -p 1113 -o pid,ppid,%mem,%cpu

  PID  PPID %MEM %CPU
 1113  6054  1.2  0.6

```

**&raquo;** `watch` execute a program periodically, showing output fullscreen

**&raquo;** `-n` Specify update interval

## Sort processes by memory, cpu
Sort all the commands with increasing cpu usage followed by memory

```bash
➜ ps -e u --sort=-%cpu,-%mem  | head -5
USER    PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
nobody 10112  1.3  6.1 2366864 1004212 ?     Sl   Nov09  46:32 /opt/google/chrome/chrome
nobody  9904  1.2  2.4 1536920 406280 ?      SLl  Nov09  40:27 /opt/google/chrome/chrome
nobody  6712  0.9  0.7 1506472 118064 ?      Ssl  Nov08  40:23 compiz
root    1313  0.9  0.5 1028880 89484 tty7    Ssl+ Nov08  38:42 /usr/lib/xorg/Xorg -core :0 -seat seat0 -auth /var/run/lightdm/root/:0 -nolisten tcp vt7 -novtswitch
```

**&raquo;** `--sort` sort by specified columns

**&raquo;** `(+/-)` + numerically increasing, - numerically decreasing

**&raquo;** `u` Display user-oriented format

## Processess view using Top command
Top command useful shortcuts

**&raquo;** `M` sort by memory usage

**&raquo;** `P` sort by cpu usage

**&raquo;** `s` change the refresh interval. Default 3.0 seconds

**&raquo;** `m` display memory in different display formats

**&raquo;** `E` display memory in different units (KB, MB, GB, etc).

**&raquo;** `c` toggle command arguments on/off

**&raquo;** `1` toggle processor level cpu usage on/off

**&raquo;** `i` toggle idle processos on/off

**&raquo;** `H` toggle between threads and tasks modes

**&raquo;** `V` toggle between forest/process tree and normal views

**&raquo;** `o` Filter by 


## References
**&raquo;** [Linux Man pages](https://www.kernel.org/doc/man-pages/)

**&raquo;** [PS Man pages](http://man7.org/linux/man-pages/man1/ps.1.html)

**&raquo;** [TOP Man pages](http://man7.org/linux/man-pages/man1/top.1.html)

**&raquo;** [PS output format](http://man7.org/linux/man-pages/man1/ps.1.html#STANDARD_FORMAT_SPECIFIERS)

**&raquo;** [Single process in tree format](https://superuser.com/questions/363169/ps-how-can-i-recursively-get-all-child-process-for-a-given-pid/822450#822450)
