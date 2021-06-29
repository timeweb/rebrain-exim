Данный репозиторий содержит конфигурационные файлы Exim, Dovecot, RoundCube, Apache2 и Nginx с помощью которых можно настроить полноценный почтовый стек MTA Exim+MDA Dovecot c RoundCube в качестве веб-интерфейса в Ubuntu 20.04 для domain.tld, где вместо domain.tld необходимо использовать желаемое доменное имя.

Предварительно для domain.tld должны быть созданы следующие DNS-записи:

```
mail.domain.tld IN A IP-адрес сервера

domain.tld IN MX mail.domain.tld

webmail.domain.tld IN A IP-адрес сервера

domain.tld IN TXT v=spf1 ip4:IP-адрес сервера -all

_dmarc.domain.tld IN TXT v=DMARC1; p=reject
```

В конфигурационных файлах и их названиях необходимо заменить domain.tld на используемое доменное имя.

В файле /roundcube/config/config.inc.php необходимо заменить password_database_roundcubemail на пароль от базы данных roundcubemail.

В файле `/etc/dovecot/passwd` необходимо заменить `password_in_md5` на пароль в формате MD5 hash.

Для возможности выпуска SSL-сертификата необходимо установить пакеты `certbot`, `python3-certbot-nginx`.

Выпуск осуществляется с помощью команды:

```bash
certbot --server https://acme-v02.api.letsencrypt.org/directory -d domain.tld -d *.domain.tld --manual --preferred-challenges dns-01 certonly
```

В качестве подтверждения, будет необходимо использовать [Проверка DNS-01](https://letsencrypt.org/ru/docs/challenge-types/).
