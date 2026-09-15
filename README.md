# FreeIPA

## 1. Запуск и проверка

```bash
docker compose up -d
docker compose ps
```

Проверка Kerberos и API:

```bash
docker exec -it freeipa-server kinit admin
docker exec freeipa-server ipa ping
curl -fsS https://freeipa.study.dev-dom.oraclesiebelcrm.com/ipa/ui/ >/dev/null
```

## 2. TLS и Certbot

Certbot хранит сертификаты в certbot-etc. Дополнительный Nginx использует
8080/8443, потому что основной FreeIPA напрямую занимает 80/443.

```bash
docker run --rm freeipa/freeipa-server:rocky-9-4.13.1 \
cat /etc/pki/tls/cert.pem > nginx/freeipa-ca-bundle.crt
chmod 644 nginx/freeipa-ca-bundle.crt
docker compose -f docker-compose.certbot.yml up -d
docker compose -f docker-compose.certbot.yml ps
docker compose -f docker-compose.certbot.yml exec certbot certbot certificates
```

## 2. Firewall

Не открывайте LDAP, Kerberos и DNS всему интернету

| Порт | Назначение | Доступ |
|---:|---|---|
| 80/tcp | HTTP redirect | Интернет |
| 443/tcp | HTTPS | доверенные сети |
| 53/tcp,udp | DNS | DNS |
| 88/tcp,udp | Kerberos | доменные клиенты |
| 389/tcp | LDAP | доменные клиенты, если нужен LDAP |
| 636/tcp | LDAPS | доменные клиенты |
| 464/tcp,udp | смена Kerberos-пароля | доменные клиенты |

## 3. Подключение Linux-клиента

На клиенте должны работать DNS и время:

```bash
sudo apt install freeipa-client
sudo ipa-client-install \
  --domain=study.dev-dom.oraclesiebelcrm.com \
  --realm=STUDY.DEV-DOM.ORACLESIEBELCRM.COM \
  --server=freeipa.study.dev-dom.oraclesiebelcrm.com \
  --mkhomedir
```

Проверка:

```bash
id admin
kinit admin
ipa ping
```

## 4. Backup

Что лучше бэкапить:

- каталог FreeIPA;
- Compose-файлы и .env;
- DNS-записи;
- сертификаты;
- список открытых портов

!!! Не считать простое копирование работающего LDAP-каталога достаточным !!!

```text
kinit admin
ipa ping
поиск пользователя
вход в Web UI
подключение тестового Linux-клиента
```
