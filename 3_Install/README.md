# Домашнее задание к уроку "Варианты установки Angie"

OS Ubuntu 24.04.1 LTS

## Установка Angie из репозитория

```sudo su```

# Установить вспомогательные пакетов
apt-get update && apt-get install -y ca-certificates curl

# Скачать открытый ключ репозитория
curl -o /etc/apt/trusted.gpg.d/angie-signing.gpg \
            https://angie.software/keys/angie-signing.gpg

# Подключить репозиторий
echo "deb https://download.angie.software/angie/$(. /etc/os-release && echo "$ID/$VERSION_ID $VERSION_CODENAME") main" \
    | tee /etc/apt/sources.list.d/angie.list > /dev/null

# Proof
cat /etc/apt/sources.list.d/angie.list
deb https://download.angie.software/angie/ubuntu/24.04 noble main

# Обновить индексы репозиториев
apt update

# Установить angie и два доп. модуля
apt install angie
apt install angie-module-njs angie-module-lua

# Proof
angie -V

Angie version: Angie/1.7.0
nginx version: nginx/1.27.1
built with OpenSSL 3.0.13 30 Jan 2024
TLS SNI support enabled
configure arguments: --prefix=/etc/angie --conf-path=/etc/angie/angie.conf --error-log-path=/var/log/angie/error.log --http-log-path=/var/log/angie/access.log --lock-path=/run/angie.lock --modules-path=/usr/lib/angie/modules --pid-path=/run/angie.pid --sbin-path=/usr/sbin/angie --http-acme-client-path=/var/lib/angie/acme --http-client-body-temp-path=/var/cache/angie/client_temp --http-fastcgi-temp-path=/var/cache/angie/fastcgi_temp --http-proxy-temp-path=/var/cache/angie/proxy_temp --http-scgi-temp-path=/var/cache/angie/scgi_temp --http-uwsgi-temp-path=/var/cache/angie/uwsgi_temp --user=angie --group=angie --with-file-aio --with-http_acme_module --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-http_v3_module --with-mail --with-mail_ssl_module --with-stream --with-stream_mqtt_preread_module --with-stream_rdp_preread_module --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-threads --feature-cache=../angie-feature-cache --with-ld-opt='-Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -Wl,-z,relro -Wl,-z,now'



ps afx
   2330 ?        Ss     0:00 angie: master process v1.7.0 #1 [/usr/sbin/angie -c /etc/angie/angie.conf]
   2331 ?        S      0:00  \_ angie: worker process #1
   2332 ?        S      0:00  \_ angie: worker process #1


2. Запуск Angie из докера

# установка докера
apt install docker.io

# запуск контейнера с Angie с пробросом порта и вынесенным за пределы каталогом со статическими файлами
docker run --name angie -v /var/www/html:/usr/share/angie/html:ro \
    -p 8800:80 -d docker.angie.software/angie:latest

# скопировать конфиги angie из докера в домашнюю директорию пользователя ubuntu
docker cp angie:/etc/angie/ /home/ubuntu/angie

# удалить старый контейнер
docker rm -f angie

# создать новый контейнер с вынесенным конфигом 
docker run --name angie -v /var/www/html:/usr/share/angie/html:ro \
    -v /home/ubuntu/angie:/etc/angie:ro -p 8800:80 -d docker.angie.software/angie:latest

# Proof
docker ps
CONTAINER ID   IMAGE                                COMMAND                  CREATED              STATUS              PORTS                                   NAMES
ada789363ade   docker.angie.software/angie:latest   "angie -g 'daemon of…"   About a minute ago   Up About a minute   0.0.0.0:8800->80/tcp, :::8800->80/tcp   angie

ps afx
  4364 ?        Sl     0:00 /usr/bin/containerd-shim-runc-v2 -namespace moby -id ada789363ade7cd6b53393e774da5956268fe70303c308ea8874ab3090c5b7d9 -address /run/contai
   4383 ?        Ss     0:00  \_ angie: master process v1.7.0 #1 [angie -g daemon off;]
   4406 ?        S      0:00      \_ angie: worker process #1
   4407 ?        S      0:00      \_ angie: worker process #1

