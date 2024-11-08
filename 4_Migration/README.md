## Домашнее задание к уроку "Миграция из Nginx в Angie"

OS Ubuntu 24.04.1 LTS

### Установить nginx на чистую виртуалку.

```
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

$ sudo nginx -v
nginx version: nginx/1.24.0 (Ubuntu)

$ ps afx | grep nginx
   6230 pts/2    S+     0:00                      \_ grep --color=auto nginx
   6215 ?        Ss     0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
   6216 ?        S      0:00  \_ nginx: worker process
   6217 ?        S      0:00  \_ nginx: worker process
   6218 ?        S      0:00  \_ nginx: cache manager process
   6219 ?        S      0:00  \_ nginx: cache loader process
```

### Перенести конфиги из домашнего задания в /etc/nginx

Получившиеся конфиги лежат в папке nginx.

### Установить angie и модули

```
$ apt install angie
$ apt install angie-module-brotli
$ apt install angie-module-rtmp
```

### Выключить автозагрузку angie
```
$ systemctl disable angie
```

### Перенос конфигурации nginx в angie:
#### Добавить в angie.conf дополнительные модули
```
load_module modules/ngx_rtmp_module.so;
load_module modules/ngx_http_brotli_filter_module.so;
load_module modules/ngx_http_brotli_static_module.so;
```

#### Скопировать из /etc/nginx в /etc/angie:
/sites-available
/sites-enabled
/snippets
proxy_params
static-avif.conf
static.conf

#### Сравнить и перенести нужное
fastcgi.conf
fastcgi_params
mime.types

#### Перенести нужное из nginx.conf в angie.conf

#### Поправить пути: заменить nginx на angie
проверить:
```
$ grep -rn '/nginx' /etc/angie/
/etc/angie/sites-available/default:18:	access_log /var/log/nginx/default.log;
/etc/angie/sites-available/default:35:		include /etc/nginx/static-avif.conf;
/etc/angie/sites-available/default:41:		include /etc/nginx/static.conf;
/etc/angie/angie.conf:100:    include /etc/nginx/sites-enabled/*;
/etc/angie/snippets/fastcgi-php.conf:8:# see: http://trac.nginx.org/nginx/ticket/321
/etc/angie/html/index.html:17:<a href="http://nginx.org/">nginx.org</a>.<br/>
/etc/angie/html/index.html:19:<a href="http://nginx.com/">nginx.com</a>.</p>

```
заменить:
```
$ find /etc/angie/ -type f -name angie.conf -exec sed --follow-symlinks -i 's|/nginx|/angie|g' {} \;

$ find /etc/angie/ -type f -name default -exec sed --follow-symlinks -i 's|/nginx|/angie|g' {} \;

$ grep -rn '/nginx' /etc/angie/
/etc/angie/snippets/fastcgi-php.conf:8:# see: http://trac.nginx.org/nginx/ticket/321
/etc/angie/html/index.html:17:<a href="http://nginx.org/">nginx.org</a>.<br/>
/etc/angie/html/index.html:19:<a href="http://nginx.com/">nginx.com</a>.</p>

```
#### Заменить символические ссылки
```
$ find /etc/angie/sites-enabled/* -type l -printf 'ln -nsf "$(readlink "%p" | sed s!/etc/nginx/sites-available!/etc/angie/sites-available!)" "$(echo "%p" | sed s!/etc/nginx/sites-available!/etc/angie/sites-available!)"\n' > script.sh

$ cat script.sh
ln -nsf "$(readlink "/etc/angie/sites-enabled/default" | sed s!/etc/nginx/sites-available!/etc/angie/sites-available!)" "$(echo "/etc/angie/sites-enabled/default" | sed s!/etc/nginx/sites-available!/etc/angie/sites-available!)"

$ chmod +x script.sh
$ ./script.sh
$ ls -la sites-enabled/
total 8
drwxr-xr-x 2 root root 4096 Nov  8 14:11 .
drwxr-xr-x 8 root root 4096 Nov  8 14:10 ..
lrwxrwxrwx 1 root root   34 Nov  8 14:11 default -> /etc/angie/sites-available/default
```
Получившиеся конфиги лежат в папке angie.
### Переключиться на angie
Проверить конфиг
```
$ angie -t
angie: the configuration file /etc/angie/angie.conf syntax is ok
angie: configuration file /etc/angie/angie.conf test is successful
```
Остановить nginx и запустить angie
```
$ sudo systemctl stop nginx && sudo systemctl start angie
$ ps afx | grep nginx
  12669 pts/2    S+     0:00                              \_ grep --color=auto nginx

$ ps afx | grep angie
  12671 pts/2    S+     0:00                              \_ grep --color=auto angie
  12661 ?        Ss     0:00 angie: master process v1.7.0 #1 [/usr/sbin/angie -c /etc/angie/angie.conf]
  12662 ?        S      0:00  \_ angie: worker process #1
  12663 ?        S      0:00  \_ angie: worker process #1
  12664 ?        S      0:00  \_ angie: cache manager process #1
  12665 ?        S      0:00  \_ angie: cache loader process #1
```
### Выключить автозагрузку nginx, включить автозагрузку angie
```
$ sudo systemctl disable nginx
Synchronizing state of nginx.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install disable nginx
Removed "/etc/systemd/system/multi-user.target.wants/nginx.service".

$ sudo systemctl enable angie
Synchronizing state of angie.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable angie
Created symlink /etc/systemd/system/multi-user.target.wants/angie.service → /usr/lib/systemd/system/angie.service.
```

```
$ systemctl status angie
● angie.service - Angie - high performance web server
     Loaded: loaded (/usr/lib/systemd/system/angie.service; enabled; preset: enabled)
     Active: active (running) since Fri 2024-11-08 14:20:31 UTC; 6min ago
       Docs: https://angie.software/en/
   Main PID: 12661 (angie)
      Tasks: 4 (limit: 2275)
     Memory: 3.6M (peak: 4.4M)
        CPU: 24ms
     CGroup: /system.slice/angie.service
             ├─12661 "angie: master process v1.7.0 #1 [/usr/sbin/angie -c /etc/angie/angie.conf]"
             ├─12662 "angie: worker process #1"
             ├─12663 "angie: worker process #1"
             └─12664 "angie: cache manager process #1"

Nov 08 14:20:31 nginx-angie systemd[1]: Starting angie.service - Angie - high performance web server...
Nov 08 14:20:31 nginx-angie systemd[1]: Started angie.service - Angie - high performance web server.

$ systemctl status nginx
○ nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: enabled)
     Active: inactive (dead)
       Docs: man:nginx(8)

Nov 08 14:20:31 nginx-angie systemd[1]: Stopping nginx.service - A high performance web server and a reverse >
Nov 08 14:20:31 nginx-angie systemd[1]: nginx.service: Deactivated successfully.
Nov 08 14:20:31 nginx-angie systemd[1]: Stopped nginx.service - A high performance web server and a reverse p>

$ systemctl mask nginx
Created symlink /etc/systemd/system/nginx.service → /dev/null.
```
