## Домашнее задание к уроку "Оптимизация производительности веб-сервисов"

ssl не настроен.
Исходные конфиги лежат в директории conf_original.

### Тест с исходным конфигом
```
oksana@wbsrv:~/Learning$ ab -n 1000 -k -c 1 http://otus.genger.ru/
This is ApacheBench, Version 2.3 <$Revision: 1903618 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking otus.genger.ru (be patient)
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


Server Software:        Angie/1.7.0
Server Hostname:        otus.genger.ru
Server Port:            80

Document Path:          /
Document Length:        59457 bytes

Concurrency Level:      1
Time taken for tests:   81.587 seconds
Complete requests:      1000
Failed requests:        0
Keep-Alive requests:    0
Total transferred:      59659000 bytes
HTML transferred:       59457000 bytes
Requests per second:    12.26 [#/sec] (mean)
Time per request:       81.587 [ms] (mean)
Time per request:       81.587 [ms] (mean, across all concurrent requests)
Transfer rate:          714.09 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        6    7   1.0      7      19
Processing:    64   75  17.2     73     442
Waiting:       47   51   9.8     50     229
Total:         70   82  17.4     80     448

Percentage of the requests served within a certain time (ms)
  50%     80
  66%     82
  75%     84
  80%     86
  90%     88
  95%     89
  98%     93
  99%    109
 100%    448 (longest request)
```

### Оптимизация
Увеличила `worker_connections` до 8192.  
Включила компиляцию регулярных выражений: `pcre_jit on` (в location есть регулярки).

Включила reuseport (оптимизация подключений по воркерам)  
Включила сброс соединений по таймауту: `reset_timedout_connection on;`

#### Настройки keepalive - оптимизация tcp-handshake
```
    keepalive_timeout  120s;
    keepalive_requests 10000;
```

#### Настройки таймаутов - защита от медленных клиентов
```
    send_timeout 10;
    client_body_timeout 10;
    client_header_timeout 10;
```
#### Оптимизация работы под большой нагрузкой - не ждать бекенд долго, а скинуть соединение как можно раньше (fail fast):
```
    proxy_connect_timeout 5;
    proxy_send_timeout    10;
    proxy_read_timeout    10;
```
#### Настройка буфера для чтения ответа от бэкенда (чтобы скинуть ответ на диск и побыстрее освободить бекенд):
```
    proxy_temp_file_write_size 64k;
    proxy_buffer_size 4k;
    proxy_buffers 64 4k;
    proxy_busy_buffers_size 32k;
```
#### Кэш открытых файловых дескрипторов - снижение кол-ва системных вызовов:
```
    open_file_cache          max=10000 inactive=60s;
    open_file_cache_valid    30s;
    open_file_cache_errors   on;
    open_file_cache_min_uses 2;
```
#### Кэш дескрипторов лог-файлов:
```
    open_log_file_cache max=100 inactive=60s min_uses=2;
```

#### Серверное кеширование

Я настраиваю кеширование корневой страницы сайта - у меня личный блог, а не новостной сайт, не ожидается частое обновление этой страницы.

Т.к. вместо proxy_pass у меня fastcgi_pass, то настройки для серверного кеширования будут такими:

angie.conf:
```
fastcgi_cache_valid 1m;
fastcgi_cache_key $scheme$host$request_uri;
fastcgi_cache_path /var/www/cache levels=1:2 keys_zone=one:10m inactive=48h max_size=800m;
```

wordpress.conf:
```
location =/ { # чтобы закешировать только корневую страницу, а не весь php
    fastcgi_cache one;
    fastcgi_cache_valid 200 1h;
    fastcgi_cache_lock on;
    fastcgi_cache_min_uses 2;
    fastcgi_ignore_headers "Cache-Control" "Expires";
    fastcgi_cache_use_stale updating error timeout invalid_header http_500;
    fastcgi_cache_background_update on;
    add_header X-FastCGI-Cache $upstream_cache_status;
}
```
#### Статическое и динамическое кеширование.
js и css сжимаю при помощи gzip и brotli.
```
gzip -9 -n -c style.css > style.css.gz
gzip -9 -n -c assets/css/editor-style.css > assets/css/editor-style.css.gz
brotli --force --quality=11 --output='style.css.br' 'style.css'
brotli --force --quality=11 --output='assets/css/editor-style.css.br' 'assets/css/editor-style.css'

```

Установила модули brotli и zstd и подключила в конфиге:
```
load_module modules/ngx_http_brotli_static_module.so;
load_module modules/ngx_http_brotli_filter_module.so;
load_module modules/ngx_http_zstd_filter_module.so;
```

в конфиге вордпресса включила gzip - статическое и динамическое сжатие:
```
    gzip on;
    gzip_static on;
    gzip_types text/plain text/css text/xml application/javascript application/json image/svg+xml application/font-ttf;
    gzip_comp_level 5;
    gzip_proxied any;
    gzip_min_length 1000;
    gzip_vary on;

```

brotli - статическое и динамическое сжатие:
```
    brotli_static on;
    brotli on;
    brotli_comp_level 5;
    brotli_types text/plain text/xml text/css application/javascript application/json image/x-icon image/svg+xml;
```

zstd - только динамическое сжатие:
```
    zstd on;
    zstd_min_length 256;
    zstd_comp_level 5;
    zstd_types text/plain text/css text/xml application/javascript application/json image/x-icon image/svg+xml;

```

#### Клиентское кеширование
Уже было настроено по-умолчанию, менять не стала.

#### HTTP/2, HTTP/3
SSL у меня не настроен, так что HTTP/2 и HTTP/3 включать смысла нет.

#### Адаптивная отдача картинок
Из jpeg картинок сгенерила avif

Настройки в wordpress.conf:
```
    location ~* \.(css|gif|ico|jpeg|jpg|js|png)$ {
        status_zone static;
        add_header Vary $vary_header;
        add_header Cache-Control $cache_control;
        try_files $uri$avif_suffix $uri$webp_suffix $uri =404;
    }
```

Настройки в angie.conf:
```
    map $http_accept $webp_suffix {
        "~*webp"  ".webp";
    }

    map $http_accept $avif_suffix {
        "~*avif"  ".avif";
        "~*webp"  ".webp";
    }

```

Оптимизированные конфиги лежат в директории conf_optimized.

### Тест после всех оптимизаций
Почему-то показал худший результат, хотя в браузере видно что и картинки отдаются avif, и сжатие работает.

```
oksana@wbsrv:~/Learning$ ab -n 1000 -k -c 1 -H "Accept-Encoding: gzip" http://otus.genger.ru/
This is ApacheBench, Version 2.3 <$Revision: 1903618 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking otus.genger.ru (be patient)
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


Server Software:        Angie/1.7.0
Server Hostname:        otus.genger.ru
Server Port:            80

Document Path:          /
Document Length:        59457 bytes

Concurrency Level:      1
Time taken for tests:   209.136 seconds
Complete requests:      1000
Failed requests:        0
Keep-Alive requests:    0
Total transferred:      59704000 bytes
HTML transferred:       59457000 bytes
Requests per second:    4.78 [#/sec] (mean)
Time per request:       209.136 [ms] (mean)
Time per request:       209.136 [ms] (mean, across all concurrent requests)
Transfer rate:          278.79 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       42   57  91.4     47    1208
Processing:   132  152  45.7    145     875
Waiting:       43   50  28.0     47     556
Total:        175  209 103.8    193    1370

Percentage of the requests served within a certain time (ms)
  50%    193
  66%    199
  75%    202
  80%    204
  90%    212
  95%    231
  98%    386
  99%    753
 100%   1370 (longest request)

```
