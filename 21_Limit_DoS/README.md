## Защита от DoS-атак, ограничение доступа

### Настройка ограничения частоты запросов

В http контексте создаю зону разделяемой памяти для ключей:

```
limit_req_zone $binary_remote_addr zone=lone:10m rate=10r/s;
```

В location ~ \.php$ активирую ограничение: позволяю в среднем не более 10 запросов в секунду со всплесками не более 10 запросов, без задержки избыточных запросов в пределах лимита:
```
limit_req zone=lone burst=10 nodelay;
limit_req_log_level error;
limit_req_status 503;
```

Проверка:
```
ab -n 100 https://otus.genger.ru/wp-admin/site-editor.php

root@nginx-angie:/etc/angie# tail -n 10 /var/log/angie/error.log
2025/01/13 10:32:43 [error] 30543#30543: *2472 limiting requests, excess: 10.020 by zone "lone", client: 79.172.72.130, server: otus.genger.ru, request: "OPTIONS /wp-json/wp/v2/navigation?_locale=user HTTP/2.0", host: "otus.genger.ru", referrer: "https://otus.genger.ru/wp-admin/site-editor.php"
2025/01/13 10:32:43 [error] 30543#30543: *2472 limiting requests, excess: 10.020 by zone "lone", client: 79.172.72.130, server: otus.genger.ru, request: "GET /wp-json/wp-block-editor/v1/navigation-fallback?_embed=true&_locale=user HTTP/2.0", host: "otus.genger.ru", referrer: "https://otus.genger.ru/wp-admin/site-editor.php"
2025/01/13 10:32:43 [error] 30543#30543: *2472 limiting requests, excess: 10.830 by zone "lone", client: 79.172.72.130, server: otus.genger.ru, request: "GET /wp-json/wp/v2/media/7?context=view&_locale=user HTTP/2.0", host: "otus.genger.ru", referrer: "https://otus.genger.ru/wp-admin/site-editor.php"
2025/01/13 10:32:45 [warn] 30543#30543: *2472 an upstream response is buffered to a temporary file /var/cache/angie/fastcgi_temp/3/01/0000000013 while reading upstream, client: 79.172.72.130, server: otus.genger.ru, request: "GET /wp-admin/site-editor.php HTTP/2.0", upstream: "fastcgi://unix:/run/php/php8.3-fpm.sock:", host: "otus.genger.ru", referrer: "https://otus.genger.ru/wp-admin/"
```

### Установка и настройка fail2ban для автоматической блокировки атакующих на основе частоты запросов

Устанавливаю fail2ban
```
apt install fail2ban
```

Настройка:
```
# скопировать файл конфигурации
cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

# в него прописать
[nginx-limit-req]
port    = http,https
enabled = true
filter  = nginx-limit-req
action  = iptables-multiport[name=ReqLimit, port="http,https", protocol=tcp]
logpath = /var/log/angie/*error.log
findtime = 600
bantime  = 7200
maxretry = 4

# перезапустить fail2ban
fail2ban-client reload

# посмотреть статус и запущенные джейлы
fail2ban-client status

# проверить статус джейла nginx-limit-req
fail2ban-client status nginx-limit-req
```

Проверка:
```
ab -n 1000 -k -c 10 https://otus.genger.ru/wp-content/

root@nginx-angie:/etc/angie# tail -n 5 /var/log/angie/error.log
2025/01/13 12:02:29 [error] 31545#31545: *30574 limiting requests, excess: 10.820 by zone "lone", client: 79.172.72.130, server: otus.genger.ru, request: "GET /wp-content/ HTTP/1.0", host: "otus.genger.ru"
2025/01/13 12:02:29 [error] 31546#31546: *30568 limiting requests, excess: 10.820 by zone "lone", client: 79.172.72.130, server: otus.genger.ru, request: "GET /wp-content/ HTTP/1.0", host: "otus.genger.ru"
2025/01/13 12:02:29 [error] 31546#31546: *30563 limiting requests, excess: 10.820 by zone "lone", client: 79.172.72.130, server: otus.genger.ru, request: "GET /wp-content/ HTTP/1.0", host: "otus.genger.ru"
2025/01/13 12:02:29 [error] 31546#31546: *30570 limiting requests, excess: 10.820 by zone "lone", client: 79.172.72.130, server: otus.genger.ru, request: "GET /wp-content/ HTTP/1.0", host: "otus.genger.ru"
2025/01/13 12:02:29 [error] 31546#31546: *30584 limiting requests, excess: 10.790 by zone "lone", client: 79.172.72.130, server: otus.genger.ru, request: "GET /wp-content/ HTTP/1.0", host: "otus.genger.ru"

root@nginx-angie:/etc/angie# fail2ban-client status nginx-limit-req
Status for the jail: nginx-limit-req
|- Filter
|  |- Currently failed:	1
|  |- Total failed:	439
|  `- File list:	/var/log/angie/mail_error.log /var/log/angie/error.log
`- Actions
   |- Currently banned:	1
   |- Total banned:	1
   `- Banned IP list:	79.172.72.130
```

### Настройка защищенного доступа с HTTP-авторизацией и ограничением доступа по IP

Установливаю apache-utils
```
apt install apache-utils
```

Создаю пользователя
```
htpasswd -c /etc/angie/htpasswd newuser
```

Закрываю доступ к angie-console - доступ с одного ip открыт без авторизаций, для всех остальных необходима авторизация:
```
location ^~ /angie-console/ {
    satisfy any;
    allow   79.172.72.130; # мой ip
    deny    all;
    auth_basic           "Please log in";
    auth_basic_user_file /etc/angie/htpasswd;
}
```

