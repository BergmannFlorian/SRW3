# SRW3 Nextcloud

## Config VM
Virtual machine name : `Nextcloud` 
Ram : `2GB`
Disk : `20GB`
Network : `Bridge` and `Host-only`
In customize hardware, remove **sound card** and **printer**

## Config Debian
System : `Debian 10`  
Langage : `english-US`  
Time format : `US`  
Keyboard : `Swiss French`
Root password : `root`
User : `nextcloud`
Password : `nextcloud`

### Install package
Run commands in root

    apt update && apt upgrade

    apt install openssh-server

    apt install -y php-{fpm,cli,json,curl,imap,gd,mysql,xml,zip,intl,imagick,mbstring,bcmath,gmp}

    apt install nginx -y

    apt install mariadb-server -y

    apt install vim

    vim /etc/mysql/my.cnf 
    
Add on the end of file:

    [mysqld]
    bind-address=0.0.0.0

## Nextcloud install
    mkdir -p /var/www

    cd /var/www

    apt install wget

    wget https://download.nextcloud.com/server/releases/latest.tar.bz2

    tar -xvf latest.tar.bz2

    rm latest.tar.bz2

    adduser nextcloud 

    chown -R nextcloud:www-data nextcloud  

### Database config
    mysql -e "CREATE DATABASE Nextcloud;"

    mysql -e "CREATE USER 'nextcloudadmin'@'' IDENTIFIED BY 'Admin123';"

    mysql -e "GRANT USAGE ON *.* TO 'nextcloudadmin'@'%';"

    mysql -e "GRANT ALL PRIVILEGES ON Nextcloud.* TO 'nextcloudadmin'@'%' WITH GRANT OPTION;"

    mysql -e "FLUSH PRIVILEGES;"

**ENABLE maintenance mode**

    cd nextcloud

    php occ maintenance:mode --on;

Change the file format of nextcloud DB to `Barracuda` :  

    mysql -e "SET GLOBAL innodb_file_format=Barracuda;"

    SELECT NAME, SPACE, FILE_FORMAT FROM INFORMATION_SCHEMA.INNODB_SYS_TABLES WHERE NAME like 'Nextcloud%'";"

    mysql -e "USE INFORMATION_SCHEMA;"

    SELECT CONCAT('ALTER TABLE ', TABLE_SCHEMA,'.', TABLE_NAME, ' ROW_FORMAT=DYNAMIC;') AS MySQLCMD FROM TABLES WHERE TABLE_SCHEMA = 'Nextcloud'";"

Put `utf8mb` on default format for nextcloud application and **Disable** maintenance mode :

    systemctl restart mariadb

    php occ config:system:set mysql.utf8mb4 --type boolean --value="true"

    php occ maintenance:mode --off

### Create ssl certificate
    mkdir -p /etc/nginx/ssl

    openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/ssl/nginx.key -out /etc/nginx/ssl/nginx.crt

    -subj "/C=CH/ST=Vaud/L=Ste-Croix/O=CPNV/OU=IT Department/CN=nextcloud.cpnv.ch"

### Config NGINX
    vim /etc/nginx/conf.d/nextcloud.conf

Add [the content](nextcloud.conf)
exit and save  the file

Go to pool of fpm and add new nextcloud pool configuration 
    
    vim /etc/php/7.3/fpm/pool.d/nextcloud.conf

Add the next content on the file
```conf
  listen = /var/run/nextcloud.sock

  listen.owner = nextcloud
  listen.group = www-data

  user = nextcloud
  group = www-data

  pm = ondemand
  pm.max_children = 30
  pm.process_idle_timeout = 60s
  pm.max_requests = 500

  env[HOSTNAME] = $HOSTNAME
  env[PATH] = /usr/local/bin:/usr/bin:/bin
  env[TMP] = /tmp
  env[TMPDIR] = /tmp
  env[TEMP] = /tmp

  request_terminate_timeout = 300
```

Install cron

    apt install cron
    vim /etc/cron.d/nextcloud

Add at end of file

    */5  *  *  *  * nextcloud php -f /var/www/nextcloud/cron.php

### Config PHP
Open `/etc/php/7.3/fpm/php.ini`, modify `memory_limit` by : `512`.

    systemcl restart php7.3-fpm

To remove the memcache error

Open `/var/www/nextcloud/config/config.php`, add `'memcache.local' => '\OC\Memcache\APCu',` after `$CONFIG = array (`.

## 3. Backup script
Thank you Gwenael for sharing the script

- create the next dirs : 
```bash
    mkdir -p /var/backups/nextcloud
```
- create the next file `/var/backups/nextcloud/backup.sh` and add the next content :  
```bash
  #!/bin/sh
  cd /var/www/nextcloud/

  php occ maintenance:mode --on

  rsync -Aavx /var/www/nextcloud/ /var/backups/nextcloud/nextcloud-dirbkp_`date +"%Y%m%d"`/
  mysqldump --single-transaction  -u nextcloud -psecret nextcloud > /var/backups/nextcloud/nextcloud-sqlbkp_`date +"%Y%m%d"`.bak

  php occ maintenance:mode --off
```

- Change the mode and the owner of backup file and restart services
```bash
chmod +x /var/backups/nextcloud/backup.sh \
&& chown -R nextcloud:www-data /var/backups/nextcloud \
&& systemctl restart cron nginx mysql php7.3-fpm
```

- Add the backup script to cron service, open `/etc/cron.d/nextcloud` and add the next line:  
```
*  22  *  *  * nextcloud php -f /var/backups/nextcloud/backup.sh
```

## 4. Tests

To test the connexion to server, on windows, open `C:\Windows\System32\drivers\etc\hosts` and add the next line, change `192.168.20.149` by your ip :  
```
	192.168.20.149  nextcloud.cpnv.ch
    10.10.10.100  nextcloud.cpnv.local
```

Open your WEB Browser and write the next URL : `nextcloud.cpnv.ch`.
Your browser ask to validate the unknow ssl certificate , validate and you can see your web site in https and he ask to log-in.

- informations to use:
    * user: nextcloud
    * password: `hello`
    * database user: nextcloudadmin
    * database passeword: Admin123
    * database name: nextcloud
    * database host: localhost

To find errors or warnings on nextcloud, go to https://nextcloud.cpnv.ch/settings/admin/overview

### POC
Network Config
- Show network card

Ngnix
- systemctl status nginx

SQL
- mysql

Nextcloud
- nextcloud.cpnv.local
- nextcloud.cpnv.ch
- nextcloud.cpnv.ch -> overview