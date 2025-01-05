## Настройка сертификатов с использованием ACME

Добавила в конфиг вордпресса
```
    location ^~/.well-known/ {
    }
```

Получила сертификат letsencrypt через certbot
```
certbot certonly --webroot -w /var/www/wordpress/ -d otus.genger.ru
```

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/otus.genger.ru/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/otus.genger.ru/privkey.pem

Добавила в конфиг
```
    listen 443 ssl;

    ssl_certificate     /etc/letsencrypt/live/otus.genger.ru/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/otus.genger.ru/privkey.pem;
```

Получение сертификата через модуль acme angie

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
