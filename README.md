Данный репозиторий содержит конфигурационные файлы Exim, Dovecot, RoundCube, Apache2 и Nginx с помощью которых можно настроить полноценный почтовый стек MTA Exim+MDA Dovecot c RoundCube в качестве веб-интерфейса в Ubuntu 20.04 для domain.tld, где вместо domain.tld необходимо использовать желаемое доменное имя.

Веб-интерфейс будет доступен по адресу:

`https://webmail.domain.tld`

Предварительно для domain.tld должны быть созданы следующие DNS-записи:

```
mail.domain.tld IN A IP-адрес сервера

domain.tld IN MX mail.domain.tld

webmail.domain.tld IN A IP-адрес сервера

domain.tld IN TXT v=spf1 ip4:IP-адрес сервера -all

_dmarc.domain.tld IN TXT v=DMARC1; p=reject

mail._domainkey.domain.tld IN TXT v=DKIM1; h=sha256; k=rsa; 
```

В конфигурационных файлах и их названиях необходимо заменить domain.tld на используемое доменное имя.

В файле /roundcube/config/config.inc.php необходимо заменить password_database_roundcubemail на пароль от базы данных roundcubemail.

В файле `/etc/dovecot/passwd` необходимо заменить `password_in_md5` на пароль в формате MD5 hash. В этом же файле осуществляется создание почтовых ящиков, по аналогии с текущим ящиком из файла.






УСТАНОВКА И НАСТРОЙКА 

Устанавливаем следующие пакеты:

`apt install exim4-daemon-heavy dovecot-core dovecot-imapd dovecot-pop3d apache2 php libapache2-mod-php php-mysql php-mbstring php7.4-gettext php-cli php-gd php7.4-opcache php-bcmath php7.4-dom mysql-server -y`





НАСТРОЙКА APACHE2

Заменяем конфигурационные файлы `/etc/apache2/ports.conf` и `/etc/apache2/sites-available/webmail.domain.tld.conf`

Активируем созданный виртуальный хост Apache2 с помощью команды:

`a2ensite webmail.domain.tld.conf`

Обновляем конфигурацию Apache2:

`systemctl reload apache2`





УСТАНОВКА И НАСТРОЙКА NGINX

`wget https://nginx.org/keys/nginx_signing.key`

`apt-key add nginx_signing.key`

Добавляем в конец файла `/etc/apt/sources.list`

`deb http://nginx.org/packages/ubuntu/ focal nginx`

`deb-src http://nginx.org/packages/ubuntu/ focal nginx`

`apt update -y`

`apt install nginx -y`

`mkdir /etc/nginx/snippets`

Заменяем конфигурационный файл `/etc/nginx/nginx.conf`

`service nginx restart`

Для возможности выпуска SSL-сертификата необходимо установить пакеты `certbot`, `python3-certbot-nginx`.

Выпуск осуществляется с помощью команды:

```
certbot --server https://acme-v02.api.letsencrypt.org/directory -d domain.tld -d *.domain.tld --manual --preferred-challenges dns-01 certonly
```

В качестве подтверждения, необходимо использовать [Проверка DNS-01](https://letsencrypt.org/ru/docs/challenge-types/).

`openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048`

Заменяем конфигурационные файлы `/etc/nginx/snippets/ssl-params.conf` и `/etc/nginx/conf.d/webmail.domain.tld.conf`

`nginx -t`

`systemctl reload nginx`





УСТАНОВКА И НАСТРОЙКА ROUNDCUBE 1

`mkdir /var/www/roundcube`

`cd /var/www/roundcube`

`wget https://github.com/roundcube/roundcubemail/releases/download/1.4.11/roundcubemail-1.4.11-complete.tar.gz`

`tar -xzvf roundcubemail-1.4.11-complete.tar.gz`

`rm roundcubemail-1.4.11-complete.tar.gz`

`mv roundcubemail-1.4.11/* /var/www/roundcube`

`rm -Rf roundcubemail-1.4.11`




НАСТРОЙКА ПАРОЛЯ ДЛЯ ПОЛЬЗОВАТЕЛЯ ROOT ДЛЯ MYSQL И СОЗДАНИЕ БАЗЫ ДАННЫХ И ПОЛЬЗОВАТЕЛЯ ДЛЯ ROUNDCUBE

`mysql_secure_installation`

`mysql -uroot -p`

`CREATE DATABASE roundcubemail;`

`CREATE USER 'roundcube'@'localhost' IDENTIFIED BY 'password_database_roundcubemail';`

`GRANT ALL ON *.* TO 'roundcube'@'localhost';`

`flush privileges;`

`quit;`





УСТАНОВКА И НАСТРОЙКА ROUNDCUBE 2

`cd /var/www/roundcube/SQL`

`mysql -uroundcube -p roundcubemail < mysql.initial.sql`

`cp /var/www/roundcube/config/config.inc.php.sample /var/www/roundcube/config/config.inc.php`

Заменяем конфигурационный файл `/var/www/roundcube/config/config.inc.php`




НАСТРОЙКА MDA DOVECOT

`groupadd -g 150 -r vmail`

`useradd -g 150 -r -u 150 vmail`

`mkdir /home/vmail`

`chown vmail:vmail /home/vmail`

`chmod u=rwx,g=rx,o= /home/vmail`

Заменяем конфигурационные файлы:

`/etc/dovecot/conf.d/10-auth.conf`
`/etc/dovecot/conf.d/auth-passwdfile.conf.ext`
`/etc/dovecot/passwd`
`/etc/dovecot/conf.d/10-mail.conf`
`/etc/dovecot/conf.d/10-ssl.conf`
`/etc/dovecot/conf.d/10-master.conf`
`/etc/dovecot/dovecot.conf`
`/etc/dovecot/conf.d/auth-system.conf.ext`

`service dovecot restart`

НАСТРОЙКА MTA EXIM

`dpkg-reconfigure exim4-config` (На первом шаге указываем internet site, на втором шаге domain.tld, на третьем шаге 0.0.0.0:, на четвертом шаге удаляем всё, дальше нажимаем Enter до конца.)

`mkdir /etc/exim4/ssl`

`cp /etc/letsencrypt/live/domain.tld/fullchain.pem /etc/exim4/ssl/fullchain.pem`

`chown root:root /etc/exim4/ssl/fullchain.pem`

`chmod 644 /etc/exim4/ssl/fullchain.pem`

`cp /etc/letsencrypt/live/domain.tld/privkey.pem /etc/exim4/ssl/privkey.pem`

`chown Debian-exim:Debian-exim /etc/exim4/ssl/privkey.pem`

`chmod 644 /etc/exim4/ssl/privkey.pem`

`echo "domain.tld" > /etc/exim4/local_domains`

Заменяем конфигурационный файл `/etc/exim4/exim4.conf.template`

`usermod -aG dovecot Debian-exim`

`chown root:vmail /var/mail/`





УСТАНОВКА OPENDKIM И НАСТРОЙКА DKIM ДЛЯ DOMAIN.TLD

`apt install opendkim-tools -y`

`mkdir /etc/exim4/dkim`

`opendkim-genkey -D /etc/exim4/dkim/ -d domain.tld -s mail --bits=1024`

`mv /etc/exim4/dkim/mail.private /etc/exim4/dkim/mail.domain.tld.private`

`mv /etc/exim4/dkim/mail.txt /etc/exim4/dkim/mail.domain.tld.public`

`cat /etc/exim4/dkim/mail.domain.tld.public` (второе значение в кавычках, котороое начинается как p=* добавляем в TXT запись для mail._domainkey.domain.tld, дополняя текущую TXT-запись)

`cd /etc/exim4/dkim`

`chmod u=rw,g=r,o= *`

`chown root:Debian-exim *`

`service exim4 reload`
