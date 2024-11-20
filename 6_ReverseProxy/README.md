## Домашнее задание к уроку "Обратный прокси (reverse proxy)"

OS Ubuntu 24.04.1 LTS
Angie уже установлен

```
root@nginx-angie:/home/ubuntu# angie -V
Angie version: Angie/1.7.0
nginx version: nginx/1.27.1
built with OpenSSL 3.0.13 30 Jan 2024
TLS SNI support enabled
configure arguments: --prefix=/etc/angie --conf-path=/etc/angie/angie.conf --error-log-path=/var/log/angie/error.log --http-log-path=/var/log/angie/access.log --lock-path=/run/angie.lock --modules-path=/usr/lib/angie/modules --pid-path=/run/angie.pid --sbin-path=/usr/sbin/angie --http-acme-client-path=/var/lib/angie/acme --http-client-body-temp-path=/var/cache/angie/client_temp --http-fastcgi-temp-path=/var/cache/angie/fastcgi_temp --http-proxy-temp-path=/var/cache/angie/proxy_temp --http-scgi-temp-path=/var/cache/angie/scgi_temp --http-uwsgi-temp-path=/var/cache/angie/uwsgi_temp --user=angie --group=angie --with-file-aio --with-http_acme_module --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-http_v3_module --with-mail --with-mail_ssl_module --with-stream --with-stream_mqtt_preread_module --with-stream_rdp_preread_module --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-threads --feature-cache=../angie-feature-cache --with-ld-opt='-Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -Wl,-z,relro -Wl,-z,now'
```

### Устанавливаем mysql и php 

```
$ apt install mysql-server-8.0

$ apt install php-fpm php-curl php-mysqli php-gd php-intl php-mbstring php-soap php-xml php-xmlrpc php-zip
```
### Заходим в mysql и создаем базу данных и пользователя для wordpress

```
$ mysql
mysql> CREATE USER 'wordpress'@'%' IDENTIFIED WITH mysql_native_password BY '1';
mysql> CREATE DATABASE wordpress;
mysql> GRANT ALL ON wordpress.* to 'wordpress'@'%';
```

### Устанавливаем wordpress
```
$ cd /tmp
$ curl -LO https://wordpress.org/latest.tar.gz
$ tar xzvf latest.tar.gz
```
### Создаем конфиг
```
$ cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
```
### Копируем все и выставляем нужные права
```
$ sudo cp -a /tmp/wordpress/. /var/www/wordpress
$ sudo chown -R www-data:www-data /var/www/wordpress
```
### Правим wp-config.php
Указываем название базы данных, имя пользователя и пароль.

### Устанавливаем wordpress
Скриншот и конфиги запущенного приложения выложены в репозиторий.
