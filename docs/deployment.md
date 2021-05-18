# Operating System
Install Ubuntu Server 20.04.2 LTS or later

# Apache
* sudo apt-get update
* sudo apt-get install apache2

## Enable Apache2 modes
* sudo a2enmod headers proxy_http xml2enc proxy ssl proxy_wstunnel rewrite

## Enable rewrite mode
* sudo nano /etc/apache2/apache2.conf/
* change to `AllowOverride All`
```
<Directory /var/www/>
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>
```

# PHP 7.2
* sudo add-apt-repository ppa:ondrej/php
* sudo apt-get update
* sudo apt-get install php7.2
* sudo apt-cache search php7.2-*
* sudo apt-get install php7.2-fpm
* sudo apt-get install php7.2-mbstring php7.2-curl php7.2-json php7.2-xml php7.2-mysql php7.2-zip
* sudo apt-get install php-pear php7.2-dev
* sudo a2enmod proxy_fcgi setenvif
* sudo a2enconf php7.2-fpm
* sudo apt-get install libapache2-mod-php7.2
* pear config-set php_ini /etc/php/7.2/apache2/php.ini
* pecl config-set php_ini /etc/php/7.2/apache2/php.ini

# Restart Apache
* sudo systemctl restart apache2

# MySQL & phpMyAdmin
* sudo apt-get install mysql-server mysql-client
* sudo apt-get install phpmyadmin

## Create Mysql User
* sudo mysql
* CREATE USER 'ward'@'localhost' IDENTIFIED BY 'password';
* GRANT ALL PRIVILEGES ON * . * TO 'ward'@'localhost';
* FLUSH PRIVILEGES;

For Mysql 8, also do:
* ALTER USER 'ward'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
* FLUSH PRIVILEGES;

# Laravel App
* sudo apt-get install git
* cd /var/www/html/
* sudo rm -r *
* git clone https://github.com/mrnivlac/up-pgh .

## Config folder permissions
* cd /var/www/
* chmod -R 775 html
* chmod -R 775 html/storage/logs
* chmod -R 775 html/bootstrap
* chown -R www-data:www-data /var/www/html/storage

## Composer
* sudo apt install composer
* sudo composer install

## Environment variables
* cd /var/www/html/
* sudo nano .env

Enter the following. Use the password you setup in mysql
```
APP_NAME=Laravel
APP_ENV=local
APP_KEY=base64:zY4l/zSS2r0RuaSX5jjdx4n3QXzbGLSPlzhxLNszUX4=
APP_DEBUG=true
APP_URL=http://localhost
LOG_CHANNEL=stack
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=up-pgh
DB_USERNAME=ward
DB_PASSWORD=password
```

# Frontend files
* Acquire the static files for the frontend (e.g. `build.zip`). Make sure you have the files that were built using the correct configuration for the api urls
  * for local server (i.e. server at http://192.168.0.50), go to https://github.com/mrnivlac/project-lifeline/releases
* unzip build.zip -d /var/www/html/lifeline-frontend/build

# Website config
* cd /etc/apache2/sites-enabled
* sudo rm -r *
* sudo nano default.conf

## For local server
```
<VirtualHost *:80>
    ServerName dev3.rxbox.app
    
    Header set Access-Control-Allow-Origin "*"

    ProxyPreserveHost On

    ProxyPass /api http://localhost:50002/
    ProxyPassReverse /api http://localhost:50002/

    ProxyPass / http://localhost:50001/
    ProxyPassReverse / http://localhost:50001/
</VirtualHost>

Listen 56002
<VirtualHost *:56002>
    Header set Access-Control-Allow-Origin "*"

    ProxyPreserveHost On

    ProxyPass / ws://localhost:57004/
    ProxyPassReverse / ws://localhost:57004/

    #SSLCertificateFile /etc/letsencrypt/live/dev2.rxbox.app/fullchain.pem
    #SSLCertificateKeyFile /etc/letsencrypt/live/dev2.rxbox.app/privkey.pem
    #Include /etc/letsencrypt/options-ssl-apache.conf
</VirtualHost>

Listen 50001
<VirtualHost *:50001>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html/lifeline-frontend/build
    
    <Directory "/var/www/html/lifeline-frontend/build">
            order allow,deny
            allow from all

            RewriteEngine on

            RewriteCond %{REQUEST_FILENAME} -s [OR]
            RewriteCond %{REQUEST_FILENAME} -l [OR]
            RewriteCond %{REQUEST_FILENAME} -d
            RewriteRule ^.*$ - [NC,L]
            RewriteRule ^(.*) /index.html [NC,L]
    </Directory>
</VirtualHost>

Listen 50002
<VirtualHost *:50002>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html/public
    
    <Directory "/var/www/html/public">
            order allow,deny
            allow from all
    </Directory>
</VirtualHost>
```

## For online server with HTTPS configuration
Use the following template. Change the server name and paths to the certifcates.
```
<VirtualHost *:80>
    ServerName dev3.rxbox.app
    RewriteEngine On
	RewriteCond %{HTTPS} off
	RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI}
</VirtualHost>

<VirtualHost *:443>
    ServerName dev3.rxbox.app

    Header set Access-Control-Allow-Origin "*"

    ProxyPreserveHost On

    ProxyPass /api http://localhost:50002/
    ProxyPassReverse /api http://localhost:50002/

    ProxyPass / http://localhost:50001/
    ProxyPassReverse / http://localhost:50001/

    SSLCertificateFile /etc/letsencrypt/live/dev2.rxbox.app/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/dev2.rxbox.app/privkey.pem
    Include /etc/letsencrypt/options-ssl-apache.conf
</VirtualHost>

Listen 56001
<VirtualHost *:56001>
    Header set Access-Control-Allow-Origin "*"

    ProxyPreserveHost On

    ProxyPass / http://localhost:57003/
    ProxyPassReverse / http://localhost:57003/

    SSLCertificateFile /etc/letsencrypt/live/dev2.rxbox.app/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/dev2.rxbox.app/privkey.pem
    Include /etc/letsencrypt/options-ssl-apache.conf
</VirtualHost>

Listen 50001
<VirtualHost *:50001>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html/lifeline-frontend/build
    
    <Directory "/var/www/html/lifeline-frontend/build">
            order allow,deny
            allow from all

            RewriteEngine on

            RewriteCond %{REQUEST_FILENAME} -s [OR]
            RewriteCond %{REQUEST_FILENAME} -l [OR]
            RewriteCond %{REQUEST_FILENAME} -d
            RewriteRule ^.*$ - [NC,L]
            RewriteRule ^(.*) /index.html [NC,L]
    </Directory>
</VirtualHost>

Listen 50002
<VirtualHost *:50002>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html/public
    
    <Directory "/var/www/html/public">
            order allow,deny
            allow from all
    </Directory>
</VirtualHost>
```

# Database procedures & data
* Login to phpmyadmin. Use the mysql credentials you setup
* import `up-pgh.sql`
* check that the procedures are present as well as thee tables in the `up-pgh` database
* by default, lifeline users will need to login using the password `magnesium`

## Changing the password for lifeline login
* Login to phpmyadmin. Use the mysql credentials you setup
* go to the `r_system_config` table in the `up-gh` table
* edit the row where rsc_configid = 1
* for the column `rsc_hashedvalue`, select the `MD5` function and enter the new password in the value field
* click go