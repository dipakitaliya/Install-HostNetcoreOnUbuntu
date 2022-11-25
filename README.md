# Install-HostNetcoreOnUbuntu

## Register Microsoft key and feed
Open a command prompt and run the following commands:
```shell
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.asc.gpg
sudo mv microsoft.asc.gpg /etc/apt/trusted.gpg.d/
wget -q https://packages.microsoft.com/config/ubuntu/18.04/prod.list 
sudo mv prod.list /etc/apt/sources.list.d/microsoft-prod.list
sudo chown root:root /etc/apt/trusted.gpg.d/microsoft.asc.gpg
sudo chown root:root /etc/apt/sources.list.d/microsoft-prod.list
```
## Install the .NET Core 3.1 Runtime
In your command prompt, run the following commands:
```shell
sudo apt-get install apt-transport-https
sudo apt-get update
sudo apt-get install aspnetcore-runtime-3.1
```
## Install the .NET SDK
In your command prompt, run the following commands:
```shell
sudo apt-get install dotnet-sdk-3.1
```

> Test the installation
```shell
dotnet --info 
```

## Install MySQL
```shell
sudo apt-get update
sudo apt-get install mysql-server mysql-client libmysqlclient-dev -y
```
> Set myslq password

```shell
sudo apt update
sudo apt install mysql-server
sudo mysql_secure_installation
sudo mysql
SELECT user,authentication_string,plugin,host FROM mysql.user;
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
FLUSH PRIVILEGES;
mysql -u root -p
```
> If you want ad user with all access
```shell
 GRANT ALL PRIVILEGES ON *.* TO 'dbroot'@'localhost' IDENTIFIED BY 'admin123' WITH GRANT OPTION;
 
 GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, SHUTDOWN, PROCESS, FILE, REFERENCES, INDEX, ALTER, SHOW DATABASES, SUPER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER, CREATE TABLESPACE, CREATE ROLE, DROP ROLE ON *.* TO `dbroot` WITH GRANT OPTION

GRANT APPLICATION_PASSWORD_ADMIN,AUDIT_ADMIN,BACKUP_ADMIN,BINLOG_ADMIN,BINLOG_ENCRYPTION_ADMIN,CLONE_ADMIN,CONNECTION_ADMIN,ENCRYPTION_KEY_ADMIN,FLUSH_OPTIMIZER_COSTS,FLUSH_STATUS,FLUSH_TABLES,FLUSH_USER_RESOURCES,GROUP_REPLICATION_ADMIN,INNODB_REDO_LOG_ARCHIVE,INNODB_REDO_LOG_ENABLE,PERSIST_RO_VARIABLES_ADMIN,REPLICATION_APPLIER,REPLICATION_SLAVE_ADMIN,RESOURCE_GROUP_ADMIN,RESOURCE_GROUP_USER,ROLE_ADMIN,SERVICE_CONNECTION_ADMIN,SESSION_VARIABLES_ADMIN,SET_USER_ID,SHOW_ROUTINE,SYSTEM_USER,SYSTEM_VARIABLES_ADMIN,TABLE_ENCRYPTION_ADMIN,XA_RECOVER_ADMIN ON *.* TO `dbroot` WITH GRANT OPTION 

```

## Install Apache as reverse proxy

Update Ubuntu packages to their latest stable versions:
```shell
sudo apt-get update
```
Install vim
```shell
sudo apt-get install vim -y
```
Install the Apache web server on Ubuntu with a single command:
```shell
sudo apt-get install apache2 -y  
```
Enable the required apache modules:
```shell
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod ssl
sudo service apache2 restart
```

## Configure Apache
Add a server’s fully qualified domain name:
```bash
sudo vim /etc/apache2/apache2.conf 
```
Add the following line to **apache2.conf**:
```bash
ServerName localhost
```
>Create a configuration file, named ***/etc/apache2/sites-available/hellomvc.conf*** ,  for the app:
```bash
cd /etc/apache2/sites-available/
sudo vim hellomvc.conf
```
>Copy and save this content to **hellomvc.conf**
```bash
<VirtualHost *:*>
    RequestHeader set "X-Forwarded-Proto" expr=%{REQUEST_SCHEME}
</VirtualHost>

<VirtualHost *:8081>
 	ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:5000/
    ProxyPassReverse / http://127.0.0.1:5000/
	
	ErrorLog /var/log/apache2/promocollabapi-error.log
    CustomLog /var/log/apache2/promocollabapi-access.log common
</VirtualHost>
```
 >For Host on Ports ***cd /etc/apache2/ports.conf*** add ports.
 ```bash
 Listen 80
 Listen 8081
 ```
 Disable the 000-default site:
 ```bash
sudo a2dissite 000-default.conf
```

Activate the hellomvc site:
 ```bash
sudo a2ensite hellomvc.conf
```
Test the configuration. If everything passes, the result should be OK.
 ```bash
sudo apachectl configtest
```
 > If you get header is not valid sytext then run 
 ```bash
 a2enmod headers
apache2ctl restart
 ```
 Restart Apache:
 ```bash
sudo service apache2 restart
```

## Monitoring the app supervisor
```bash
sudo apt-get install supervisor
```
>Create a file /etc/supervisor/conf.d/hellomvc.conf
```bash
sudo vim /etc/supervisor/conf.d/hellomvc.conf
```
Add this content to hellomvc.conf
```bash
[program:dotnettest]
command=/usr/bin/dotnet /var/aspnetcore/hellomvc.dll --urls "http://*:5000"
directory=/var/aspnetcore/
autostart=true
autorestart=true
stderr_logfile=/var/log/hellomvc.err.log
stdout_logfile=/var/log/hellomvc.out.log
environment=ASPNETCORE_ENVIRONMENT=Production
user=www-data
stopsignal=INT
```
Now we start and stop Supervisor and watch/tail its logs to see our app startup!
```bash
sudo service supervisor stop
sudo service supervisor start
sudo tail -f /var/log/supervisor/supervisord.log
# and the application logs if you like
sudo tail -f /var/log/hellomvc.out.log 
```
Output
```bash
2018-07-13 12:15:08,102 INFO supervisord started with pid 9314
2018-07-13 12:15:09,105 INFO spawned: 'dotnettest' with pid 9317
2018-07-13 12:15:10,336 INFO success: dotnettest entered RUNNING state, process has stayed up for > than 1 secon
 ```

## Check WWW Port listen
```bash
egrep -w '(80|22|443)/tcp' /etc/services	
sudo lsof -i -P -n | grep LISTEN
```
## My Public IP

```bash
curl http://ifconfig.io
```

## You should enable proxy. Run a command:
```bash
 sudo a2enmod proxy
 apache2ctl restart
```

## SSL CSR create apache Ubuntu
```js
mkdir ~/ssl
cd ~/ssl
openssl req -new -newkey rsa:2048 -nodes -keyout promocollab_key_tld.key -out promocollab_csr_tld.csr
```


## Binding DNS

 > Records

| Type | Name | Value | TTL    
| ------ | ------ | ------ | ------ |
| A | @ | 	3.20.162.240 | 600 seconds 
|CNAME |www |@ |	1 Hour
|CNAME	|_domainconnect|	_domainconnect.gd.domaincontrol.com|	1 Hour
| NS |	@|	ns77.domaincontrol.com |	1 Hour
| NS |	@|	ns78.domaincontrol.com	| 1 Hour
|SOA	|@|	Primary nameserver: ns77.domaincontrol.com...|	1 Hour
|CNAME	|api|	ec2-3-20-162-240.us-east-2.compute.amazonaws.com...|	1 Hour

> Change in APACHE2 .conf file
 ```bash
 <VirtualHost *:80>
    ServerName api.nftmining.com
    ServerAlias api.nftmining.com

        RewriteEngine On
        RewriteRule (.*) https://api.nftmining.com/ [R=301,L]

    DocumentRoot /var/www/html/cg-backend-laravel/public
    <Directory /var/www/html/cg-backend-laravel/public>
        Options -Indexes +FollowSymLinks
        AllowOverride All
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/api-gotham.com-error.log
    CustomLog ${APACHE_LOG_DIR}/api-gotham.com-access.log combined
</VirtualHost>
<VirtualHost *:443>
ServerName api.nftmining.com
#ServerAlias api.nftmining.com

 DocumentRoot /var/www/html/cg-backend-laravel/public
 <Directory /var/www/html/cg-backend-laravel/public>
     Options -Indexes +FollowSymLinks
     AllowOverride All
 </Directory>

 ErrorLog ${APACHE_LOG_DIR}/api-gotham.com-error.log
 CustomLog ${APACHE_LOG_DIR}/api-gotham.com-access.log combined


SSLEngine on
SSLCertificateFile /root/ssl/__nftmining_com.crt
SSLCertificateKeyFile /root/ssl/nftmining_tld.key
SSLCertificateChainFile /root/ssl/__nftmining_com.ca-bundle
</VirtualHost>
 
 ```


## httpd init
 > init /etc/supervisord.d/migrate.ini
 ```bash
 [program:migrationAPI]
command=/usr/bin/dotnet /var/git/cg-rpgbackend-wct/MigrationToolAPI/bin/Debug/net6.0/MigrationToolAPI.dll --urls "http://*:5000"
process_name=migrationAPIProcess ; process_name expr (default %(program_name)s)
numprocs=1
directory=/var/git/cg-rpgbackend-wct/MigrationToolAPI/
;logfile=/var/log/supervisor/migrate.log
;umask=022
;priority=999
;autostart=true
;autorestart=true
;startsecs=10
;startretries=3
;exitcodes=0,2
;stopsignal=QUIT
;stopwaitsecs=10
;user=chrism
;redirect_stderr=true
stdout_logfile=/var/log/supervisor/migrate.log
stdout_logfile_maxbytes=10MB
stdout_logfile_backups=10
stdout_capture_maxbytes=10MB
;stdout_events_enabled=false
;stderr_logfile=/a/path
;stderr_logfile_maxbytes=1MB
;stderr_logfile_backups=10
;stderr_capture_maxbytes=1MB
;stderr_events_enabled=false
;environment=A=1,B=2
;serverurl=AUTO
 ```
