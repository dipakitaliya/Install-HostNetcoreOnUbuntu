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

<VirtualHost *:8082>
 	ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:5001/
    ProxyPassReverse / http://127.0.0.1:5001/
	
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

