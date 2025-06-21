# Cacti
How to install and Configure Cacti on Ubuntu 20.04


Here is a properly formatted and organized `README.md` file for GitHub, covering **all the steps** with **clean Markdown syntax**, including **highlighted Linux commands** and **section headers** for readability:

---

# 📊 Cacti Network Monitoring Tool Installation Guide on Ubuntu 22.04+

This guide walks you through installing and configuring the **Cacti** network monitoring tool on an Ubuntu server. Cacti requires LAMP, SNMP, RRDTool, and other dependencies.

---

## ✅ Step 1: Getting Started

Start by updating your system packages:

```bash
sudo apt-get update && sudo apt-get upgrade -y
```

Install required dependencies:

```bash
sudo apt-get install snmp php-snmp rrdtool librrds-perl unzip curl git gnupg2 -y
```

---

## 🧱 Step 2: Install LAMP Stack

Install Apache, MariaDB, PHP, and necessary PHP extensions:

```bash
sudo apt-get install apache2 mariadb-server php php-mysql libapache2-mod-php php-xml php-ldap php-mbstring php-gd php-gmp -y
```

Edit the Apache PHP configuration file:

```bash
sudo nano /etc/php/8.1/apache2/php.ini
```

Change the following lines:

```
memory_limit = 512M
max_execution_time = 60
date.timezone = Asia/Dhaka
```

Now edit the CLI PHP configuration file:

```bash
sudo nano /etc/php/8.1/cli/php.ini
```

Update the same lines:

```
memory_limit = 512M
max_execution_time = 60
date.timezone = Asia/Dhaka
```

Restart Apache:

```bash
sudo systemctl restart apache2
```

---

## 🛢️ Step 3: Configure MariaDB Server

Edit MariaDB server configuration:

```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

Add/modify under `[mysqld]` section:

```
collation-server = utf8mb4_unicode_ci
max_heap_table_size = 128M
tmp_table_size = 64M
join_buffer_size = 4M
sort_buffer_size = 4M
innodb_file_format = Barracuda
innodb_large_prefix = 1
innodb_buffer_pool_size = 1G
innodb_flush_log_at_timeout = 3
innodb_read_io_threads = 32
innodb_write_io_threads = 16
innodb_io_capacity = 5000
innodb_io_capacity_max = 10000
innodb_doublewrite = OFF
```

Restart MariaDB:

```bash
sudo systemctl restart mariadb
```

Login to MariaDB and create the Cacti database:

```bash
sudo mysql -u root -p
```

Inside MariaDB shell:

```sql
CREATE DATABASE cacti;
GRANT ALL ON cacti.* TO 'cactiuser'@'localhost' IDENTIFIED BY 'cactiuser';
FLUSH PRIVILEGES;
EXIT;
```

Import timezone data:

```bash
sudo mysql mysql < /usr/share/mysql/mysql_test_data_timezone.sql
```

Then grant timezone access:

```bash
sudo mysql -u root -p
```

Inside MariaDB shell:

```sql
GRANT SELECT ON mysql.time_zone_name TO 'cactiuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

## 🌱 Step 4: Install and Configure Cacti

Download and extract Cacti:

```bash
wget https://www.cacti.net/downloads/cacti-latest.tar.gz
tar -zxvf cacti-latest.tar.gz
sudo mv cacti-1* /var/www/html/cacti
```

Import the Cacti SQL template:

```bash
sudo mysql cacti < /var/www/html/cacti/cacti.sql
```

Edit the Cacti config file:

```bash
sudo nano /var/www/html/cacti/include/config.php
```

Ensure these settings are correct:

```php
$database_type     = 'mysql';
$database_default  = 'cacti';
$database_hostname = 'localhost';
$database_username = 'cactiuser';
$database_password = 'cactiuser';
$database_port     = '3306';
```

Create log file and set permissions:

```bash
sudo touch /var/www/html/cacti/log/cacti.log
sudo chown -R www-data:www-data /var/www/html/cacti/
sudo chmod -R 775 /var/www/html/cacti/
```

Create a Cacti cron job:

```bash
sudo nano /etc/cron.d/cacti
```

Add:

```cron
*/5 * * * * www-data php /var/www/html/cacti/poller.php > /dev/null 2>&1
```

---

## 🌐 Step 5: Configure Apache for Cacti

Create Apache config:

```bash
sudo nano /etc/apache2/sites-available/cacti.conf
```

Add the following:

```apache
Alias /cacti /var/www/html/cacti

<Directory /var/www/html/cacti>
    Options +FollowSymLinks
    AllowOverride None
    <IfVersion >= 2.3>
        Require all granted
    </IfVersion>
    <IfVersion < 2.3>
        Order Allow,Deny
        Allow from all
    </IfVersion>

    AddType application/x-httpd-php .php

    <IfModule mod_php.c>
        php_flag magic_quotes_gpc Off
        php_flag short_open_tag On
        php_flag register_globals Off
        php_flag register_argc_argv On
        php_flag track_vars On
        php_value mbstring.func_overload 0
        php_value include_path .
    </IfModule>

    DirectoryIndex index.php
</Directory>
```

Enable site and restart Apache:

```bash
sudo a2ensite cacti
sudo systemctl restart apache2
```

---

## 🌍 Step 6: Final Setup in Browser

Open your browser and navigate to:

```
http://your-server-ip/cacti
```

Use the default credentials to log in:

* **Username:** `admin`
* **Password:** `admin`

Follow the setup wizard to complete the installation.

![look at this](/photo/photo1.png width="500px" heigth="500px")

---

## 🎉 You're Done!

Cacti is now installed and ready to monitor your network devices. Happy graphing!

---
