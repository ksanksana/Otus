## Настройка HTTPS для веб-сервисов

### Получение сертификата Let's encrypt через модуль acme

в angie.conf:
```
    resolver 127.0.0.53 ipv6=off;
    acme_client rsa   https://acme-v02.api.letsencrypt.org/directory key_type=rsa;
    acme_client ecdsa https://acme-v02.api.letsencrypt.org/directory;

```

в wordpress.conf:
```
    acme                rsa;
    acme                ecdsa;

    ssl_certificate     $acme_cert_rsa;
    ssl_certificate_key $acme_cert_key_rsa;
    ssl_certificate     $acme_cert_ecdsa;
    ssl_certificate_key $acme_cert_key_ecdsa;

```

### Настройка основных параметров HTTPS в Angie

Генерим файл с параметрами Диффи-Хеллмана

```
openssl dhparam -out dhparam.pem 4096
```

В angie.conf:

```
    ##
    # SSL Settings
    ##

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_dhparam /etc/angie/dhparam.pem;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-CHACHA20-POLY1305;
    ssl_prefer_server_ciphers off;

```

### Оптимизируем восстановление сессий

В angie.conf:

кеширование сессий tls на стороне сервера:
```
    # ssl session caching 
    # server-side
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 28h;

```

на стороне клиента:
```
    # client-side
    ssl_session_tickets on;

```

###  Автоматическая переадресация с HTTP на HTTPS

в wordpress.conf:
```
server {
    listen      80 reuseport;
    server_name otus.genger.ru;
    return 301 https://otus.genger.ru$request_uri;
}
```

### Настройка заголовка HSTS

HSTS - заставить браузер использовать HTTPS на долгое время

в wordpress.conf:
```
add_header Strict-Transport-Security max-age=31536000;
```

### Настройка HTTP/2 и HTTP/3

в wordpress.conf
```
http2 on;
http3 on;

listen 443 quick reuseport;

add_header Alt-Svc 'h3=":443"; ma=86400';

quic_gso on;
quic_bpf on;
quic_retry off;
```

### Тестирование корректности конфигурации с помощью внешнего сервиса

https://testtls.com/otus.genger.ru/443
https://www.ssllabs.com/ssltest/analyze.html?d=otus.genger.ru


