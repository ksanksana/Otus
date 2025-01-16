# Тема: Реализация отказоустойчивой инфраструктуры веб-приложения на базе веб-сервера Angie
## Структура проекта:

- балансировщик
- backend1
- backend2
- mysql

<img src=/Project/scheme.jpg width=800 />

### Балансировщик
<a href=/Project/balancer>Конфиг</a>

Установлены Angie, angie-console-light, модули brotli и zstd.

### Backend-1, Backend-2
<a href=/Project/backends>Конфиг</a>

Установлены Angie и Wordpress.

#### Устанавливаем wordpress
```
$ cd /tmp
$ curl -LO https://wordpress.org/latest.tar.gz
$ tar xzvf latest.tar.gz
```
##### Создаем конфиг
```
$ cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
```
##### Копируем все и выставляем нужные права
```
$ mkdir /var/www
$ mkdir /var/www/wordpress

$ sudo cp -a /tmp/wordpress/. /var/www/wordpress
$ sudo chown -R www-data:www-data /var/www/wordpress
```
##### Правим wp-config.php
Указываем название базы данных, имя пользователя и пароль.

#### Устанавливаем mysql

```
$ apt install php-fpm php-curl php-mysqli php-gd php-intl php-mbstring php-soap php-xml php-xmlrpc php-zip
```

## db
устанавливаю mysql
```
apt install mysql-server-8.0
```

Для того, чтобы к базе можно было обращаться с внешнего сервера, надо в /etc/mysql/mysql.conf.d/mysqld.cnf закомментировать строчку
```
#bind-address           = 127.0.0.1
```
и перезагрузить mySQL service следующей командой:
```
sudo /etc/init.d/mysql restart
```
### Заходим в mysql и создаем базу данных и пользователя для wordpress

```
$ mysql
mysql> CREATE USER 'wordpress'@'%' IDENTIFIED WITH mysql_native_password BY '1';
mysql> CREATE DATABASE wordpress;
mysql> GRANT ALL ON wordpress.* to 'wordpress'@'%';
```

## Нагрузочное тестирование
```
oksana@wbsrv:~/.ssh$ ab -n 1000 -k -c 1 -H "Accept-Encoding: gzip" https://oksana-deeva.ru/
This is ApacheBench, Version 2.3 <$Revision: 1903618 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking oksana-deeva.ru (be patient)
Completed 100 requests
Completed 200 requests
Completed 300 requests
Completed 400 requests
Completed 500 requests
Completed 600 requests
Completed 700 requests
Completed 800 requests
Completed 900 requests
Completed 1000 requests
Finished 1000 requests


Server Software:        Angie/1.8.1
Server Hostname:        oksana-deeva.ru
Server Port:            443
SSL/TLS Protocol:       TLSv1.3,TLS_AES_256_GCM_SHA384,256,256
Server Temp Key:        X25519 253 bits
TLS Server Name:        oksana-deeva.ru

Document Path:          /
Document Length:        58641 bytes

Concurrency Level:      1
Time taken for tests:   151.052 seconds
Complete requests:      1000
Failed requests:        500
   (Connect: 0, Receive: 0, Length: 500, Exceptions: 0)
Keep-Alive requests:    0
Total transferred:      58887500 bytes
HTML transferred:       58600500 bytes
Requests per second:    6.62 [#/sec] (mean)
Time per request:       151.052 [ms] (mean)
Time per request:       151.052 [ms] (mean, across all concurrent requests)
Transfer rate:          380.71 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       14   23   4.1     23      87
Processing:    83  128  28.5    121     337
Waiting:       77  117  28.0    111     330
Total:        100  151  29.5    144     362

Percentage of the requests served within a certain time (ms)
  50%    144
  66%    155
  75%    163
  80%    169
  90%    188
  95%    204
  98%    229
  99%    268
 100%    362 (longest request)
```
