## Балансировка нагрузки (HTTP)

### Равномерная балансировка (round robin).
```
upstream backend_rr {
    zone upstream-backend-rr 256k;
    server 127.0.0.1:9000 weight=2;
    server 127.0.0.1:9001;
    server 127.0.0.1:9002;
    server 127.0.0.1:9003;
}

server {
   location ^~ /rr/ {
        access_log  /var/log/angie/balance_rr_access.log  balance;
        status_zone balance_rr;
        proxy_pass http://backend_rr;
    }
}
```

### Балансировка по хэшу с использованием переменных.
```
upstream backend_hash {
    zone upstream-backend-hash 256k;
    hash $scheme$request_uri;
    server 127.0.0.1:9000;
    server 127.0.0.1:9001;
    server 127.0.0.1:9002;
    server 127.0.0.1:9003;
}

server {
    location ^~ /hash/ {
        access_log  /var/log/angie/balance_hash_access.log  balance;
        status_zone balance_hash;
        proxy_pass http://backend_hash;
    }
}
```

#### Проверка работоспособности
```
ab -n 100 -k -c 1 http://otus.genger.ru/hash/
ab -n 100 -k -c 1 http://otus.genger.ru/hash/a=1
ab -n 100 -k -c 1 http://otus.genger.ru/hash/a=2
ab -n 100 -k -c 1 http://otus.genger.ru/hash/a=3
ab -n 100 -k -c 1 http://otus.genger.ru/hash/a=4
```
<img src="/16_Balancing/hash.png" width=400/>

### Произвольная балансировка (random).
```
upstream backend_random {
    zone upstream-backend-random 256k;
    random;
    server 127.0.0.1:9000;
    server 127.0.0.1:9001;
    server 127.0.0.1:9002 weight=3;
    server 127.0.0.1:9003;
}

server {
    location ^~ /random/ {
        access_log  /var/log/angie/balance_random_access.log  balance;
        status_zone balance_random;
        proxy_pass http://backend_random;
    }
}
```

#### Проверка работоспособности
```
ab -n 100 -k -c 1 http://otus.genger.ru/random/
```
<img src="/16_Balancing/random.png" width=400/>

### Вариант конфигурации с резервным бэкендом и с отключением одного из бэкендов (на примере rr балансировки).

```
upstream backend_rr {
    zone upstream-backend-rr 256k;
    server 127.0.0.1:9000 weight=2;     # white
    server 127.0.0.1:9001;              # blue
    server 127.0.0.1:9002 backup;       # green
    server 127.0.0.1:9003 down;         # gold
}
```
#### Проверка работоспособности
```
ab -n 100 -k -c 1 http://otus.genger.ru/rr/
```
<img src="/16_Balancing/rr.png" width=400/>
