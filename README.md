# ss-panel V3

Let's talk about cat.  Based on [LightFish](https://github.com/Pongtan/LightFish).

[![Build Status](https://travis-ci.org/orvice/ss-panel.svg?branch=master)](https://travis-ci.org/orvice/ss-panel) [![Coverage Status](https://coveralls.io/repos/github/orvice/ss-panel/badge.svg?branch=master)](https://coveralls.io/github/orvice/ss-panel?branch=master) [![Join the chat at https://gitter.im/orvice/ss-panel](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/orvice/ss-panel?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

[Releases](https://plus.google.com/communities/112308980947577664041) |[Follow on Trello](https://trello.com/b/dr62AtYI/ss-panel) | [Google+](https://plus.google.com/communities/112308980947577664041)

## About

Please visit [releases pages](https://github.com/orvice/ss-panel/releases) to download ss-panel.

## Requirements

* PHP 5.6 or newer
* Web server with URL rewriting
* MySQL

## Supported Server

* [shadowsocks manyuser](https://github.com/mengskysama/shadowsocks/tree/manyuser)
* [shadowsocks-go mu](https://github.com/orvice/shadowsocks-go)


## Install LAMP on Ubuntu 16.04 LTS 

### Step 1 Install Apache
```
$ sudo apt-get update
$ sudo apt-get install apache2
$ sudo nano /etc/apache2/apache2.conf
```

Add the following line into /etc/apache2/apache2.conf, change the server_domain_or_IP accordingly.
```
ServerName server_domain_or_IP
```

check for syntax errors
```
$ sudo apache2ctl configtest
```

You should see this
```
Output
Syntax OK
```

Restart Apache to get effective
```
$ sudo systemctl restart apache2
```
Check if it is working, you should see the Ubuntu logo and apache2 default page.
Before accessing your apache server, you probably need to adjust the firewall to allow web traffic, but we skip this here.
```
http://your_server_IP_address
```
### Step 2 Install MySQL
```
$ sudo apt-get install mysql-server
```
Sever will ask you to select and confirm a password for the MySQL "root" user.

### Step 3 Install PHP
```
$ sudo apt-get install php libapache2-mod-php php-mcrypt php-mysql
```

Modify the way that Apache serves files when a directory is requested. 
```
$ sudo nano /etc/apache2/mods-enabled/dir.conf
```

Make Apache look for an index.php file first. Put .php on the first.
```
<IfModule mod_dir.c>
    DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```

Restart the Apache web server in order to apply your changes
```
$ sudo systemctl restart apache2
```
### Step 4 Install PHP Modules

We need to install more modules later.
```
$ sudo apt-get install php-cli
```
### Step 5 Test PHP
Create php test page.
```
$ sudo nano /var/www/html/info.php
```

put everything below into the file.
```
<?php
phpinfo();
?>
```

test it in your browser.
```
http://your_server_IP_address/info.php
```

more detail please refer to 
https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-16-04

### Step 6 Install phpMyAdmin and components.
```
$ sudo apt-get install phpmyadmin php-mbstring php-gettext
$ sudo phpenmod mcrypt
$ sudo phpenmod mbstring
```

restart Apache for your changes
```
$ sudo systemctl restart apache2
```

test your phpmyadmin
```
http://domain_name_or_IP/phpmyadmin
```

1. Create a user "ss-panel" same as yours in the .env we will talk about later.
2. Import db.sql into mysql, this database will be named as "ss-panel" too, automatically.
3. Grand data usage privilege for "ss-panel".


## Install ss-panel
### Step 1 Download
use shadowsocks proxy to download v3.4.5 tar.gz
```
$ export http_proxy='192.168.0.7:1080' && wget https://github.com/orvice/ss-panel/archive/v3.4.5.tar.gz
```

Unpack
```
$ tar -xvf v3.4.5.tar.gz
```

Change directory to ~/ss-panel-3.4.5/
```
$ cd ss-panel-3.4.5/
```

download composer through shadowsocks proxy or any other proxy 
```
$ export HTTPS_PROXY='192.168.0.7:1080' && curl -sS https://getcomposer.org/installer | php
```

### Step 2 Install composer
```
$ php composer.phar install
```

You will be experiencing lots of errors as below.
```
PHP error: “The zip extension and unzip command are both missing, skipping.”

Loading composer repositories with package information
Updating dependencies (including require-dev)
    Failed to download psr/log from dist: The zip extension and unzip command are both missing, skipping.
The php.ini used by your command-line PHP is: /etc/php/7.0/cli/php.ini
    Now trying to download from source
```

Install necessary components.
```
$ sudo apt install zip unzip php7.0-zip
$ sudo apt-get install php-xml
$ sudo apt-get install php-mbstring
$ sudo apt-get install php-gd
$ sudo apt-get install php-curl
```

Try install composer again, this time shoule be OK.
```
$ php composer.phar install
```

### Step 3 Link ss-panel to home directory
```
$ sudo ln -s ~/ss-panel-3.4.5/ /var/www/html/ss
```
### Step 4 Enabling mod_rewrite
```
$ sudo a2enmod rewrite
$ sudo service apache2 restart
```

A .htaccess file is located at ~/ss/public, allow changes in the .htaccess file. 
We will need to set up and secure a few more settings before we can begin.
```
$ sudo nano /etc/apache2/sites-enabled/000-default.conf
```

put the following line right below this line<VirtualHost *:80>
```
<Directory /var/www/html>
                Options Indexes FollowSymLinks MultiViews
                AllowOverride All
                Order allow,deny
                allow from all
</Directory>
```

To put these changes into effect, restart Apache.
```
$ sudo service apache2 restart
```

### Step 5 Set Environment and privileges
```
$ cp .env.example .env
```

then edit .env

put read and write privilege on storage, otherwise you will encount Slim Application Error
```
$ chmod -R 777 storage
```

### Step 6 Import Data Base

Import the sql to you mysql database by phpmyadmin. 
```
http://domain_name_or_IP/phpmyadmin
```

1. Create a user "ss-panel" same as yours in the .env on step 2.
2. Import db.sql into mysql, this database will be named as "ss-panel" too, automatically.
3. grand data usage privilege for "ss-panel".


### Step 7 Change DocumentRoot directory

Change DocumentRoot directory to ss-panel, after this you are still able to access phpmyadmin.
If you wish to do some changes on mySQL, just access phpmyadmin by http://domain_name_or_IP/phpmyadmin in your browser.
```
$ sudo nano /etc/apache2/sites-enabled/000-default.conf
```

Change the DocumentRoot point to path /var/www/html/ss/public
```
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html/ss/public
```

To put these changes into effect, restart Apache.
```
$ sudo service apache2 restart
```


### Step 8 Start ss-panel

add default administrator account.
input e-mail address and new password for ss-panel administrator.
```
$ php xcat createAdmin
```

Open a browser and check your ss-panel working or not, login and make some changes. 
```
http://domain_name_or_IP/
```

## Install shadowsocks-rm
### Step 1 clone from github
```
$ git clone -b manyuser https://github.com/myfingerhurt/shadowsocks-rm.git
```

### Step 2 Install pip
```
$ sudo apt install python-pip
```

Install cymysql for shadowsocks-manyuser
```
$ pip install cymysql
```
### Step 3 Final
Star shadowsocks-manyuser process with custom DNS.
```
$ sudo python shadowsocks-rm/shadowsocks/servers.py --dns-server 8.8.8.8
```

track the log
```
$ tail -f /var/log/shadowsocks.log
```

## Trouble shoot

If you get this message, you can safely ignore it.
```
You are using pip version 8.1.1, however version 9.0.1 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
```
Or you can update anyway.
```
$ pip install --upgrade pip
```

if you see this, that's your muKey in .env which does not match with API_PASS in config.py, match them, and try again.
```
HTTPError: HTTP Error 405: Not Allowed
HTTPError: HTTP Error 401: Unauthorized
```

