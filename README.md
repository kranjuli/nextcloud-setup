# Guide to setup Nextcloud on Raspberry Pi

Guides to install NextCloud server on Raspberry Pi, to upload pictures oder videos from smartphone to NextCloud automatically

## Initial Setup

Install NextCloud auf Raspberry Pi

1. Upgrade OS in Raspberry Pi  

   ```
   sudo apt update
   sudo apt upgrade -y
   sudo apt dist-upgrade -y
   sudo apt autoremove -y
   ```

2. Install nextcloud's requirements

   ```
   sudo apt install apache2 php libapache2-mod-php mariadb-server php-mysql php-xml php-zip php-gd php-curl php-mbstring php-bcmath php-json php-ldap wget unzip -y
   ```

3. Setup MariaDB (MySQL)

   ```
   sudo mysql

   CREATE DATABASE nextcloud;
   CREATE USER 'your_nextcloud_user'@'localhost' IDENTIFIED BY 'your_password';
   GRANT ALL PRIVILEGES ON nextcloud.* TO 'your_nextcloud_user'@'localhost';
   FLUSH PRIVILEGES;
   EXIT;

   sudo systemctl status mariadb
   ```

4. Download and install Nextcloud

   ```
   cd /var/www/
   sudo wget https://download.nextcloud.com/server/releases/nextcloud-31.0.4.tar.bz2  # Ersetze die Version, wenn nötig
   sudo tar -xjf nextcloud-*.tar.bz2
   sudo chown -R www-data:www-data nextcloud/
   sudo chmod -R 755 nextcloud/
   ```

5. Configure Apache

   Edit  Apache config file for Nextcloud:

   ```
    sudo nano /etc/apache2/sites-available/nextcloud.conf
   ```

   Insert following config:

   ```
   <VirtualHost *:80>
     # ServerAdmin admin@your_domain.com
     DocumentRoot /var/www/nextcloud
     ServerName ip_oder_domain_of_nextcloud(localhost)

     <Directory /var/www/nextcloud>
        Options +FollowSymLinks
        AllowOverride All
        Require all granted
     </Directory>

   </VirtualHost>
   ```

   ```
   sudo a2ensite nextcloud.conf
   sudo systemctl reload apache2

   sudo a2dissite 000-default.conf

   sudo a2enmod rewrite headers env dir mime
   sudo systemctl restart apache2
   ```

6. Setup Nextcloud in Browser

   In browser call IP of Raspberry Pi (e.g. http://raspberrypi.local or http://your_IP).

   Configure Nextcloud with installation assistant.
   - input your Datanbase information: Datanbase name nextcloud, username, password.
   - continue with the installation and allow Nextcloud to check the database connection.

7. Install redis for caching 

   ```bash
   sudo apt install redis-server php-redis -y

   # check status of redis
   sudo systemctl status redis-server

   # enable and start of redis-server still not enable
   sudo systemctl enable redis-server
   sudo systemctl start redis-server

   # test redis
   redis-cli ping
   # answer must be PONG

   # config in nextcloud
   sudo nano /var/www/nextcloud/config/config.php

   # insert following config into config.php
   #'memcache.locking' => '\\OC\\Memcache\\Redis',
   #'memcache.local' => '\\OC\\Memcache\\Redis',
   #'redis' => [
   #     'host' => 'localhost',
   #     'port' => 6379,
   #     'timeout' => 0.0,
   #  ],

   sudo usermod -aG redis www-data
   ```

8. Configure OPCache

   ```bash
   sudo nano /etc/php/8.2/apache2/php.ini

   #[opcache]
   #opcache.enable=1
   #opcache.enable_cli=1
   #opcache.memory_consumption=128
   #opcache.interned_strings_buffer=8
   #opcache.max_accelerated_files=10000
   #opcache.revalidate_freq=60
   #opcache.save_comments=1
   #opcache.jit=off
   ```

## Access from iPhone

Install Nextcloud App on iPhone and configure

## Move nextcloud to external SSD

> [!NOTE]
> Linux supports only SSD with ext4 format.
> Command to format the SSD with ext4:: sudo mkfs.ext4 /dev/sda1

### Nextcloud application

```bash
# show connected devices
lsbk

# create mount point
sudo mkdir /mnt/ssd
sudo mount /dev/sda1 /mnt/ssd

# persistence mounting
sudo nano /etc/fstab
# insert line below into /etc/fstab
# /dev/sda1 /mnt/ssd ext4 defaults 0 0

# stop apache2
sudo systemctl stop apache2

# mv /var/www/nextcloud to /mnt/ssd
sudo mv /var/www/nextcloud /mnt/ssd/

sudo chown -R www-data:www-data /mnt/ssd/nextcloud

# adjust config
sudo nano /mnt/ssd/nextcloud/config/config.php
# insert line below into config.php
'datadirectory' => '/mnt/ssd/nextcloud_data'

cd /var/www
sudo /mnt/ssd/nextcloud nextcloud 

sudo systemctl start apache2
# only at first mounting
sudo -u www-data php /var/www/nextcloud/occ maintenance:repair
```

### nextcloud's database

```bash
sudo systemctl stop mysql
# or sudo systemctl stop mariadb

mkdir /mnt/ssd/mysql
sudo chown -R mysql:mysql /mnt/ssd/mysql

# copy database
sudo rsync -av /var/lib/mysql/ /media/ssd/mysql/

sudo chown -R mysql:mysql /mnt/ssd/mysql

rm -rf /var/lib/mysql

cd /var/lib
ln -s /mnt/ssd/mysql mysql

sudo systemctl start mysql
sudo systemctl status mysql
```

## Use local hostname for Raspberry Pi

Install `avahi` to use local hostname e.g. <your_nextcloud_hostname> instead of ip of the raspberry pi

```bash
sudo apt install avahi -y

systemctl status avahi-daemon

# set hostname for raspberry pi

sudo hostnamectl set-hostname <your_nextcloud_hostname>
```

## Backup nextcloud from SSD after flashing of SD card
* Create Mounting
  ```bash
  # create mount point to ssd
  lsblk
  sudo mkdir /mnt/ssd
  # ssd = sda1
  mount /dev/sda1 /mnt/ssd
  ```
* Mysql:
  ```bash
  sudo systemctl stop mysql
  # or sudo systemctl stop mariadb

  sudo mv /var/lib/mysql /var/lib/mysql.bak

  # create symlink for mysql
  cd /var/lib
  sudo ln -s /mnt/ssd/mysql mysql

  sudo chown -R mysql:mysql /var/lib/mysql

  sudo systemctl start mysql
  sudo systemctl status mysql
  ```

* Nextcloud app:
  ```bash
  # create symlink for nextcloud app
  sudo systemctl stop apache2
  cd /var/www
  sudo ln -s /mnt/ssd/nextcloud nextcloud
  sudo chown -R www-data:www-data nextcloud

  sudo systemctl stop apache2

  # run occ scan 
  sudo -u www-data php /var/www/html/nextcloud/occ files:scan --all
  ```
