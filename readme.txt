######### this iinstallation for amazon linux 2 server #####################

#!/bin/bash
# WordPress Auto Installer for Amazon Linux 2
# Run as: sudo bash install_wordpress.sh

# Update system
yum update -y

# Install Apache
yum install -y httpd
systemctl start httpd
systemctl enable httpd

# Install required packages
amazon-linux-extras enable php8.0
yum install -y php php-mysqlnd php-fpm php-json php-gd php-xml php-mbstring php-curl php-intl php-zip
systemctl restart httpd

# Install MariaDB
sudo dnf update
dnf search mariadb
sudo dnf install mariadb105-server
mariadb -V
systemctl start mariadb
systemctl enable mariadb

# Secure MariaDB (auto)
mysql -u root
ALTER USER 'root'@'localhost' IDENTIFIED BY 'StrongRootPass!123';
DELETE FROM mysql.user WHERE User='';
DROP DATABASE IF EXISTS test;
DELETE FROM mysql.db WHERE Db='test' OR Db='test\_%';
FLUSH PRIVILEGES;

# Create WordPress DB and user
CREATE DATABASE wordpress;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'StrongWPpass!123';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;


# Download WordPress
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xvzf latest.tar.gz
rsync -avP wordpress/ /var/www/html/

# Configure wp-config.php
cd /var/www/html
cp wp-config-sample.php wp-config.php

sed -i "s/database_name_here/${DBNAME}/" wp-config.php
sed -i "s/username_here/${DBUSER}/" wp-config.php
sed -i "s/password_here/${DBPASS}/" wp-config.php

# Set permissions
chown -R apache:apache /var/www/html/
chmod -R 755 /var/www/html/

# Restart Apache
systemctl restart httpd

echo "=========================================="
echo " WordPress installation completed!"
echo " Visit: http://<your-ec2-public-ip>"
echo " DB User: $DBUSER"
echo " DB Pass: $DBPASS"
echo "=========================================="


###############################################

#for new user  wordpress and exiting wordpress server
#step 1
#we need to download wordpress for new user 

mkdir /var/www/html/test_user
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xvzf latest.tar.gz
sudo rsync -avP wordpress/ /var/www/html/test_user/

#step 2
#create  db for new user
mysql -u root -p     #put on root password
CREATE DATABASE test_wp;
CREATE USER 'test_user'@'localhost' IDENTIFIED BY 'StrongPass!456';
GRANT ALL PRIVILEGES ON test_wp.* TO 'test_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;

#step3
#update wp_config.php with new test_db user
cp wp-config-sample.php wp-config.php\

#take refrence with this line
sudo vi wp-config.php

define('DB_NAME', 'test_wp');
define('DB_USER', 'test_user');
define('DB_PASSWORD', 'StrongPass!456');
define('DB_HOST', 'localhost');
