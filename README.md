  

# Magento 2 with Nginx and Letsencrypt on Ubuntu 18.04

## Prerequisites
-   Ubuntu 18.04
-   Root privileges
## Summary
1.  Install Nginx on Ubuntu 18.04
2.  Install and Configure PHP-FPM 7.1
3.  Install and Configure MySQL Server
4.  Install and Configure Magento 2
5.  Install PHP Composer
6.  Download Magento 2
7.  Install Magento Components
8.  Generate SSL Letsencrypt
9.  Configure Nginx Virtual Host for Magento
10.  Magento Post-Installation


## Step 1 - Install Nginx on Ubuntu 18.04 LTS
#### 1.1 Update & Upgrade all packages.
```bash
sudo apt update
sudo apt upgrade
```
#### 1.2 Install the Nginx web server
```bash
sudo apt install nginx -y
```
#### 1.3 Start the Nginx service and enable it to launch every time at system boot.

```bash
systemctl start nginx
systemctl enable nginx
```

## Step 2 - Install and Configure PHP-FPM 7.1
List of PHP extensions needed for Magento 2 installation:
-   bc-math
-   ctype
-   curl
-   dom
-   gd, ImageMagick 6.3.7 (or later) or both
-   intl
-   mbstring
-   mcrypt
-   hash
-   openssl
-   PDO/MySQL
-   SimpleXML
-   soap
-   spl
-   libxml
-   xsl
-   zip
-   json
-   iconv
#### 2.1 Install the 'software-properties-common' package and add the 'ondrej/php' repository using commands below
```bash
sudo apt install software-properties-common -y
sudo add-apt-repository ppa:ondrej/php -y
```
#### 2.2 Install PHP-FPM 7.1 with all extensions needed.
```bash
sudo apt install php7.1-fpm php7.1-mcrypt php7.1-curl php7.1-cli php7.1-mysql php7.1-gd php7.1-xsl php7.1-json php7.1-intl php-pear php7.1-dev php7.1-common php7.1-mbstring php7.1-zip php7.1-soap php7.1-bcmath -y
```

#### 2.3 Check the PHP version and installed extensions
```bash
php -v
php -me
```
#### 2.4 Configure the php.ini file for the PHP-FPM and PHP-CLI
edit 
```bash
vim /etc/php/7.1/fpm/php.ini
vim /etc/php/7.1/cli/php.ini
```
Change the value of those lines as below.
```bash
memory_limit = 512M
max_execution_time = 180
zlib.output_compression = On
```
restart the PHP-fpm service and enable it to launch every time at system boot

```bash
systemctl restart php7.1-fpm
systemctl enable php7.1-fpm
```

check the service

```bash
netstat -pl | grep php
```
## Step 3 - Install and Configure MySQL Server

Magento 2.1.2 or later require the MySQL 5.7.x
 we will install latest MySQL server 5.8 on the Ubuntu 18.04 system.
#### 3.1 Install MySQL 5.8

```bash
sudo apt install mysql-server mysql-client -y
```

#### 3.2  Start the MySQL service and enable it to launch every time at system boot.

```bash
systemctl start mysql
systemctl enable mysql
```

#### 3.3  Configure the MySQL root password 
use 'mysql_secure_installation' command

```bash
mysql_secure_installation
```

#### 3.4 Create a new Mysql database for Magento installation.
Login to the MySQL shell using the root user.

```bash
mysql -u root -p
```
create the new mysql database and new user.

```bash
create database magentodb;
create user magentouser@localhost identified by 'Magento0463@#';
grant all privileges on magentodb.* to magentouser@localhost identified by 'Magento0463@#';
flush privileges;
```
## Step 4 - Install and Configure Magento 2

In this step, we will install Magento 2.2.4 latest version from Github repository. We will install the PHP composer for installing the Magento components, download Magento from Github repository, configure Nginx virtual host for Magento, and install Magento using the web-based post installation.
#### 4.1 Install PHP Composer
```bash
sudo apt install composer -y
```
check the composer version installed

```bash
composer -V
```
#### 4.2 Download Magento 2
Go to the '/var/www' directory and download the Magento archive source code from Github using wget command.

```bash
cd /var/www/
wget https://github.com/magento/magento2/archive/2.2.4.tar.gz
```
Now extract the Magento archive file and rename the directory to 'magento2'.

```
tar -xf 2.2.4.tar.gzmv magento2-2.2.4/ magento2/
```
The Magento source code has been downloaded, and the '/var/www/magento2' directory will be the web root for the Magento site.

#### 4.3 Install Magento Components

Install Magento components using the PHP composer. Go to the 'magento2' directory and install all the PHP components needed by Magento using the 'composer' comman:d.

```bash
cd /var/www/magento2
composer install -v
```

#### 4.4  Generate SSL Letsencrypt
Install the Letsencrypt:
```bash
sudo apt install letsencrypt -y
```
After the installation is complete, stop the nginx service.
generate the SSL certificates for the domain name using certbot:

```
certbot certonly --standalone -d domaine.magento.test
```

The Letsencrypt SSL certificate files will be generated to the '/etc/letsencrypt/live' directory.

 #### 4.5 Configure Nginx Virtual Host
 
 Go to the '/etc/nginx/sites-available' directory and create new virtual host file 'magento'.
```bash
cd /etc/nginx/sites-available/vim magento
```
Paste the following configuration:

```bash
upstream fastcgi_backend {
        server  unix:/run/php/php7.1-fpm.sock;
}

server {
    listen 80;
    listen [::]:80;
    server_name magento.hakase-labs.pw;
    return 301 https://$server_name$request_uri;
}

server {

        listen 443 ssl;
        server_name magento.hakase-labs.pw;

        ssl on;
        ssl_certificate /etc/letsencrypt/live/magento.hakase-labs.pw/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/magento.hakase-labs.pw/privkey.pem;

        set $MAGE_ROOT /var/www/magento2;
        set $MAGE_MODE developer;
        include /var/www/magento2/nginx.conf.sample;
}
```

activate the virtual host by creating the symbolic link for the Magento virtual host file to the 'sites-enabled' directory.

```bash
ln -s /etc/nginx/sites-available/magento /etc/nginx/sites-enabled/
```
Test nginx configuration:
```bash
nginx -t
```
Now restart the PHP-FPM and Nginx service.

```
systemctl restart php7.1-fpm
systemctl restart nginx
```
And change the owner of Magento web-root directory to the 'www-data' user and group.

```bash
chown -R www-data:www-data /var/www/magento2/
```
The nginx virtual host for Magento has been added.

#### 4.6 Magento Post-Installation

Open the web browser and type the Magento URL. Mine is:

https://domaine.magento.test/

And _Follow The Wizard_. 

Disable write access for the '/var/www/magento2/app/etc' directory.

```bash
sudo chmod -w /var/www/magento2/app/etc
```