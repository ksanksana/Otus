## Балансировка почтовых сервисов

### Конфиг http/mail

### auth скрипт

### Установка mailcow

https://docs.mailcow.email/getstarted/install/#installation-via-paketmanager-plugin

```
apt install docker-compose-v2 git

su
umask
# 0022 # <- Verify it is 0022

cd /opt
git clone https://github.com/mailcow/mailcow-dockerized
cd mailcow-dockerized

./generate_config.sh

# поменять MAILCOW_HOSTNAME
nano mailcow.conf

# Перед этим убираем 80 порт из конфигов Angie
docker compose pull
docker compose up -d

```

#### Настройка data/conf/postfix/extra.cf:
https://docs.mailcow.email/manual-guides/Postfix/u_e-postfix-unauthenticated-relaying/

```
mynetworks = 127.0.0.0/8 172.22.1.0/24 192.168.122.0/24
```

### Настройка data/conf/rspamd/local.d/options.inc:
```
local_addrs = [127.0.0.0/8, 172.22.1.0/24, 192.168.122.0/24];
```
```
docker compose restart postfix-mailcow rspamd-mailcow
```
