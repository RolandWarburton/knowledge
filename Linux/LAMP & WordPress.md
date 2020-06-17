# LAMP & Wordpress

### Basic setup

On the newly provisioned server. Create a user and enable a firewall, allowing ssh access.

```none
adduser roland
usermod -aG sudo roland
```

Create firewall rules for OpenSSH.

```none
ufw app list
ufw allow OpenSSH
ufw enable
ufw status
```

Copy ssh keys to the new server from your own computer to make logging in easier and secure.

```none
ssh-copy-id -i ~/.ssh/id_nopass_rsa roland@123.45.678.910
```

Install apache 2 software

```none
sudo apt update
sudo apt install apache2
```

Add firewall profile to enable in UFW.

```none
sudo vim /etc/ufw/applications.d/apache2-utils.ufw.profile
```

```none
[Apache Full]
title=<title>
description=<description>
ports=80/tcp|443/tcp
```

```none
sudo ufw app update "Apache Full"
sudo ufw allow "Apache Full"
```

Test the connection by navigating to your servers ip in a browser.

```none
curl ifconf.me <- get the ip of the website
http://123.45.678.910
```

Install mariaDB which replaces MySQL

```none
sudo apt install mariadb-server
```

Secure the database

```none
sudo mysql_secure_installation
```

Create a database

```sql
CREATE DATABASE example_database;

GRANT ALL ON example_database.* TO 'example_user'@'localhost' IDENTIFIED BY 'mypassword' WITH GRANT OPTION;

-- reload privilages
FLUSH PRIVILEGES;
```

Log into the new database with the new user

```none
mariadb -u example_user -p
or
mariadb -u example_user --password='mypassword'
```

See the databases you have created above

```sql
SHOW DATABASES;

-- Switch to database
USE example_database;
SHOW TABLES;
```

Install php

```none
sudo apt install php libapache2-mod-php php-mysql
```

Prefer PHP files over .html files as the index

```none
sudo vim /etc/apache2/mods-enabled/dir.conf

<IfModule mod_dir.c>
    DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```

Reload apache2

```none
sudo systemctl reload apache2
sudo systemctl status apache2
```

Create virtual host/s

```none
sudo mkdir /var/www/\[domain_name\]
sudo chown -R $USER:$USER /var/www/\[domain_name\]

sudo vim /etc/apache2/sites-available/\[domain_name\].conf
```

```none
/etc/apache2/sites-available/\[domain_name\].conf

<VirtualHost *:80>
    ServerName your_domain
    ServerAlias www.your_domain 
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/your_domain
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Enable the virtual host

```none
sudo a2ensite your_domain
```

Disable the default host

```none
sudo a2dissite 000-default
```

Test the config

```none
sudo apache2ctl configtest
```

Point domain to IP (namecheap)

Under "Advanced DNS".

| Type     | Host | Val |
| -------- | ---- | --- |
| A record | www  | ip  |
| A record | @    | ip  |

### LetsEncrypt on Apache

Install certbot (avaliable on debian repos already!)

```none
sudo apt install certbot
sudo apt install python-certbot
```

Vertify the servername in your websites apache config file.

```none
sudo vim /etc/apache2/sites-available/\[domain_name\].conf
```

Obtain the cert with certbot.

```none
sudo certbot --apache -d your_domain.com -d www.your_domain.com
```

Email: Your personal email\
(A)gree with the ToS\
(N)o to the mailing list\
option 2: redirect http -> https

The location of the certificate will be stored at

```none
/etc/letsencrypt/live/\[domain_name\]/fullchain.pem
```

The location of the key file will be stored at

```none
/etc/letsencrypt/live/\[domain_name\]/privkey.pem
```

The directory `/etc/letsencrypt` should be securely backed up as account credentials (keys) are stored here.

Do a dry run renewal to ensure that everything is working

```none
sudo certbot renew --dry-run
```

### Setting up wordpress

Once completing the above sections, you are ready to create a wordpress skeleton.

Create a new user for WP. If no admin account exists, create one using your default root account.

```none
sudo mariadb -u root
```

```sql
CREATE USER 'admin'@'localhost' IDENTIFIED BY 'mypassword';
GRANT ALL PRIVILEGES ON *.* TO 'admin'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

Then create the WP user while logged in as a privilaged account (root or admin).

```sql
CREATE USER 'wordpress'@'localhost' IDENTIFIED BY 'mypassword';
GRANT ALL ON wordpress.* TO 'wordpress'@'localhost' IDENTIFIED BY 'mypassword';
FLUSH PRIVILEGES;
```

Create a database for wordpress

```sql
CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
```

Now log in as the wordpress user and view the wordpress database to verify that it was created.

```none
mariadb -u wordpress -p
```

```sql
SHOW DATABASES;
```

At this stage nothing new has been performed thats WP specific. Just like above, php can connect to your frontend. For example using the following test code from [digital ocean](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mariadb-php-lamp-stack-on-debian-10). You can see a basic table connection working.

```none
maria -u example_user -p
```

```sql
CREATE TABLE example_database.todo_list (
	item_id INT AUTO_INCREMENT,
	content VARCHAR(255),
	PRIMARY KEY(item_id)
);

INSERT INTO example_database.todo_list (content) VALUES ("My 1st important item");
INSERT INTO example_database.todo_list (content) VALUES ("My 2nd important item");
```

```php
<?php
$user = "example_user";
$password = "password";
$database = "example_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
```

Next we need to extend the php with common/popular wordpress specific plugins.

```none
sudo apt update
sudo apt install php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip
sudo systemctl restart apache2
```

According to digital ocean, "Currently, the use of .htaccess files is disabled. WordPress and many WordPress plugins use these files extensively for in-directory tweaks to the web server’s behavior."

To enable `.htaccess` you need to enable it on a per site basis

```none
sudo vim /etc/apache2/sites-avaliable/\[domain_name\].conf
```

And append the following `<Directory>` block to the bottom.

```none
<VirtualHost *:80>
	// stuff here
</VirtualHost>

<Directory /var/www/\[domain_name\]/>
    AllowOverride All
</Directory>
```

Next enable the rewrite module to usilize the WP permalink feature.

```none
sudo a2enmod rewrite
```

Finally check if your config was set up correctly and reload.

```none
sudo apache2ctl configtest
sudo systemctl restart apache2
```

Next you need to download the latest tarball of wordpress. Put this in your home directory for now.

```none
mkdir wordpress
cd wordpress
curl -O https://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
```

Create a skeleton `.htaccess` inside the wordpress folder you just extracted.

```none
touch wordpress/.htaccess
```

Next copy over the sample WP config file to bootstrap the WP setup.

```none
cp wordpress/wp-config-sample.php wordpress/wp-config.php
```

Next, Digital Ocean recommends creating an upgrade directory that is owned by a non root user to avoid permissions issues.

```none
mkdir -p wordpress/wp-content/upgrade
```

Now yeet the entire prepared WP directory over into your `/var/www/\[domain_name\] directory. Make sure that all the documents are on the base of the directory and not still insdie the wordpress directory.

```none
cp -a wordpress/* /var/www/0x4.host/
```

Apache uses the www-data user and group to run and so the files should be chowned to this group for wordpress to perform updates etc.

```none
sudo chown -R www-data:www-data /var/www/0x4.host
```

Next, Digital Ocean provides two find commands to change file permissions

Change all directories to 750

```none
sudo find /var/www/\[domain_name\]/ -type d -exec chmod 750 {} \;
```

Change all files to 640

```none
sudo find /var/www/\[domain_name\]/ -type f -exec chmod 640 {} \;
```

Next get some good random secret keys for wp-config.php

```none
curl -s https://api.wordpress.org/secret-key/1.1/salt/
```

paste the results from the above curl to the following file, replacing the old ones.

```none
vim /var/www/\[domain_name\]/wp-config.php
```

Next change the database name, username, etc to correct values for the wordpress database you created earlier. Leave DB_COLATE empty.

```none
DB_NAME: 'change me'

DB_USER: 'change me'

DB_PASSWORD: 'change me'

DB_HOST: 'change me'

DB_CHARSET: 'change me'
```

Next navigate to your site and complete the install process.

Pick a strong password! and log in, you are now on the dash. Good luck!
