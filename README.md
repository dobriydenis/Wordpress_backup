# Backup Wordpress
Скрипт для backup одного сайта на Wordpress, файлы, база данных, конфигурационные файлы Nginx, SSL сертификаты

---

## Предварительно подготовимся

`mkdir -p /backup/Temp`

`cd /backup`

`nano backup.sh`

---

И вставляем туда следующее содержимое:

```
#!/bin/bash

TODAY=$(date '+%Y%m%d')
TEMP_DIR=/backup/Temp/
BACKUP_NAME="site.ru"
# Директория, где будет храниться бэкап
BACKUP_DIR=/backup/

# Параметры базы данных
DB_NAME="db_name"
DB_USER="db_user"
DB_PASS="db_password"

# Пути к файлам сайта, конфигам nginx и SSL сертификатам
SITE_PATH=/var/www/site.ru
NGINX_CONF_PATH=/etc/nginx

#Раскоментировать, если нужны сертификаты
#SSL_CERT_PATH=/etc/letsencrypt

echo "Starting Backup..."

mkdir $TEMP_DIR

# Backup базы данных (mysql)
mysqldump -u $DB_USER -p$DB_PASS $DB_NAME > $TEMP_DIR/database.sql

# Backup файлов сайта
tar --exclude="updraft" -zcf $TEMP_DIR/files.tar.gz -C $SITE_PATH .

# Backup директории nginx
tar -zcf $TEMP_DIR/nginx_conf.tar.gz -C $NGINX_CONF_PATH .

# Backup SSL сертификатов
#tar -zcf $TEMP_DIR/ssl_cert.tar.gz -C $SSL_CERT_PATH .

# Создание архива
tar -zcf $BACKUP_DIR/$BACKUP_NAME-$TODAY.tar.gz -C $TEMP_DIR .

# Удаление временных файлов
rm -Rf $TEMP_DIR

echo "Backup Complete [$(du -sh $BACKUP_DIR/$BACKUP_NAME-$TODAY.tar.gz | awk '{print $1}')]"
#Удаление бэкапов старше 20 дней
echo "Removing backups older than 20 days..."
find $BACKUP_DIR -name "$BACKUP_NAME-*.tar.gz" -mtime +20 -exec rm {} \;
echo "Old backups removed."
```

---
### Cron (каждое воскресенье в 00:00)

`crontab -e`

`0 0 * * 0 /backup/backup.sh`
