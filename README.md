# Лабораторная 1

### Написать сервис, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова. Файл и слово должны задаваться в /etc/sysconfig

## Создадим файл с конфигурацией сервиса в /etc/sysconfig

```
[root@centos7-vm sysconfig]\# vim watchlog
WORD="ALERT"
LOG=/var/log/watchlog.log
```
## Создаем файл /var/log/watchlog.log

```
[root@centos7-vm sysconfig]\# touch /var/log/watchlog.log
[root@centos7-vm sysconfig]\# ls -l /var/log
...
-rw-r--r--. 1 root   root         0 Dec 11 19:51 watchlog.log
...
```

## Создадим скрипт
```
[root@centos7-vm sysconfig]\# touch /opt/watchlog.sh
[root@centos7-vm sysconfig]\# vim /opt/watchlog.sh

#!/bin/bash

WORD=$1
LOG=$2
DATE=`date`

if grep $WORD $LOG &> /dev/null
then
    logger "$DATE: founded needed word $WORD"
else
    exit 0
fi
```

## Создадим юнит для сервиса
```
[root@centos7-vm ~]\# touch watchlog.service
[root@centos7-vm ~]\# vim watchlog.service
[root@centos7-vm ~]\# cat watchlog.service
[Unit]
Description=My watchlog service

[Service]
Type=oneshot
EnvironmentFile=/etc/sysconfig/watchlog
ExecStart=/opt/watchlog.sh $WORD $LOG
```

## Создадим юнит для таймера
```
[root@centos7-vm ~]\# touch watchlog.timer
[root@centos7-vm ~]\# vim watchlog.timer
[root@centos7-vm ~]\# cat watchlog.timer
[Unit]
Description=Run watchlog script every 30 second

[Timer]
\# Run every 30 second
OnUnitActiveSec=30
Unit=watchlog.service

[Install]
WantedBy=multi-user.target
```

## Запустить и убедиться в результате
```
[root@centos7-vm system]\# systemctl start watchlog.timer
[root@centos7-vm system]\# tail -f /var/log/messages
Dec 11 21:02:30 centos7-vm atop: PRC | sys    0.05s | user   0.02s | \#proc     82 | \#zombie    0 | \#exit      0 |
Dec 11 21:02:30 centos7-vm atop: CPU | sys       0% | user      0% | irq       0% | idle    100% | wait      0% |
Dec 11 21:02:30 centos7-vm atop: CPL | avg1    0.01 | avg5    0.04 | avg15   0.05 | csw      463 | intr     310 |
Dec 11 21:02:30 centos7-vm atop: MEM | tot   991.2M | free  372.6M | cache 406.7M | buff    2.0M | slab  106.0M |
Dec 11 21:02:30 centos7-vm atop: SWP | tot     2.0G | free    2.0G |              | vmcom 334.1M | vmlim   2.5G |
Dec 11 21:02:30 centos7-vm atop: PID SYSCPU USRCPU  VGROW  RGROW  RDDSK  WRDSK  THR S CPUNR  CPU CMD
Dec 11 21:02:30 centos7-vm atop: 30821  0.03s  0.01s     0K     0K     0K     0K    1 R     0   0% atop
Dec 11 21:02:30 centos7-vm atop: 1051  0.01s  0.01s     0K     8K     0K     0K    1 S     0   0% systemd-journa
Dec 11 21:02:30 centos7-vm atop: 17000  0.01s  0.00s     0K     0K     0K     0K    1 S     0   0% kworker/0:1
Dec 11 21:02:30 centos7-vm atop: 1471  0.00s  0.00s     0K     4K     0K     4K    3 S     0   0% rsyslogd
...
```

### Из epel установить spawn-fcgi и переписать init-скрипт на unit-файл. Имя сервиса должно также называться
## Установка spawn-fcgi и пакеты
```
yum install epel-release -y && yum install spawn-fcgi php php-cli mod_fcgid httpd -y
Loaded plugins: fastestmirror
Determining fastest mirrors
epel/x86_64/metalink                                                                                                                   |  29 kB  00:00:00
 * base: mirror.docker.ru
...

[root@centos7-vm system]\# cat /etc/sysconfig/spawn-fcgi
\# You must set some working options before the "spawn-fcgi" service will work.
\# If SOCKET points to a file, then this file is cleaned up by the init script.
\#
\# See spawn-fcgi(1) for all possible options.
\#
\# Example :
SOCKET=/var/run/php-fcgi.sock
OPTIONS="-u apache -g apache -s $SOCKET -S -M 0600 -C 32 -F 1 -P /var/run/spawn-fcgi.pid -- /usr/bin/php-cgi"

[root@centos7-vm system]\# cat /etc/systemd/system/spawn-fcgi.service
[Unit]
Description=Spawn-fcgi startup service
After=network.target

[Service]
Type=simple
PIDFile=/var/run/spawn-fcgi.pid
EnvironmentFile=/etc/sysconfig/spawn-fcgi
ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS
KillMode=process

[Install]
WantedBy=multi-user.target

[root@centos7-vm system]# systemctl start spawn-fcgi
[root@centos7-vm system]# systemctl status spawn-fcgi
● spawn-fcgi.service - Spawn-fcgi startup service
   Loaded: loaded (/etc/systemd/system/spawn-fcgi.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2019-12-11 21:12:26 UTC; 7s ago
 Main PID: 17275 (php-cgi)
   CGroup: /system.slice/spawn-fcgi.service
           ├─17275 /usr/bin/php-cgi
           ├─17276 /usr/bin/php-cgi
           ├─17277 /usr/bin/php-cgi
           ├─17278 /usr/bin/php-cgi
           ├─17279 /usr/bin/php-cgi
           ├─17280 /usr/bin/php-cgi
           ├─17281 /usr/bin/php-cgi
           ├─17282 /usr/bin/php-cgi
           ├─17283 /usr/bin/php-cgi
           ├─17284 /usr/bin/php-cgi
           ├─17285 /usr/bin/php-cgi
           ├─17286 /usr/bin/php-cgi
           ├─17287 /usr/bin/php-cgi
           ├─17288 /usr/bin/php-cgi
           ├─17289 /usr/bin/php-cgi
           ├─17290 /usr/bin/php-cgi
           ├─17291 /usr/bin/php-cgi
           ├─17292 /usr/bin/php-cgi
           ├─17293 /usr/bin/php-cgi
           ├─17294 /usr/bin/php-cgi
           ├─17295 /usr/bin/php-cgi
           ├─17296 /usr/bin/php-cgi
           ├─17297 /usr/bin/php-cgi
           ├─17298 /usr/bin/php-cgi
           ├─17299 /usr/bin/php-cgi
           ├─17300 /usr/bin/php-cgi
           ├─17301 /usr/bin/php-cgi
           ├─17302 /usr/bin/php-cgi
           ├─17303 /usr/bin/php-cgi
           ├─17304 /usr/bin/php-cgi
           ├─17305 /usr/bin/php-cgi
           ├─17306 /usr/bin/php-cgi
           └─17307 /usr/bin/php-cgi

Dec 11 21:12:26 centos7-vm systemd[1]: Started Spawn-fcgi startup service.
```

### Дополнить юнит-файл apache httpd возможностью запустить несколько инстансов сервера с разными конфигурациями
## Шаблоны
```
[root@centos7-vm system]# cat httpd.service
[Unit]
Description=The Apache HTTP Server
After=network.target remote-fs.target nss-lookup.target
Documentation=man:httpd(8)
Documentation=man:apachectl(8)

[Service]
Type=notify
EnvironmentFile=/etc/sysconfig/httpd-%l
ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND
ExecReload=/usr/sbin/httpd $OPTIONS -k graceful
ExecStop=/bin/kill -WINCH ${MAINPID}
KillSignal=SIGCONT
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

## Окружение файла
```
[root@centos7-vm system]\# cat /etc/sysconfig/httpd-first
OPTIONS="-f conf/first.conf"
[root@centos7-vm system]\# cat /etc/sysconfig/httpd-second
OPTIONS="-f conf/second.conf"

[root@centos7-vm system]\# cat /etc/sysconfig/conf/first.conf
PidFile /var/run/httpd-first.pid
Listen 80
[root@centos7-vm system]\# cat /etc/sysconfig/conf/second.conf
PidFile /var/run/httpd-second.pid
Listen 8000
```
