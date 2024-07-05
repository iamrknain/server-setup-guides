## How to create a http-apache server to host Matomo:

- First make a server (vm, vps, etc).
- Login to it/ access remote server.

### Lets use azure ubuntu vm
- access vm as :  `sudo ssh -i path/to/ssh_key.pem user_name@public_ip`
```
change file permission as: sudo chmod 600 matomovmkey.pem
check permissions:  ls -l matomovm.pem
```
### Initial installation and setup:
```
sudo apt update
sudo apt upgrade -y
```
Other requirements:
```
sudo apt install apache2 -y
sudo apt install mysql-server -y
```
Secure mysql installation: `sudo mysql_secure_installation`
```
sudo apt install php libapache2-mod-php php-mysql php-xml php-curl php-gd php-cli php-common php-mbstring php-zip -y
```
Now downloading source code of required package using wget:
```
wget https://builds.matomo.org/matomo-latest.zip

unzip matomo-latest.zip
```
Install unzip if required: `sudo apt install unzip -y`

Now move the extracted files to the : `sudo mv matomo /var/www/html/`

Ensure the web server has the correct permissions to access the Matomo files:
```
sudo chown -R www-data:www-data /var/www/html/matomo
sudo chmod -R 755 /var/www/html/matomo
```
Configure apache for serving package over internet and overriding default file serving as: 
- Create a new file for serving package : `sudo nano /etc/apache2/sites-available/matomo.conf`
- Add the following configuration for http:
```
<VirtualHost *:80>
    ServerAdmin ravikumar117211@gmail.com
    DocumentRoot /var/www/html/matomo
    ServerName public_ip
    <Directory /var/www/html/matomo>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Enable the new configuration and rewrite module:
```
sudo a2ensite matomo.conf
sudo a2enmod rewrite
```
Restart apache: `sudo systemctl restart apache2`

### Configure MySQL: 
- Log into MySQL : `sudo mysql -u root -p and enter password (mysql_password) and then create db`
- Create MySQL db as: 
```
CREATE DATABASE db_name;
CREATE USER 'user_name'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON db_name.* TO 'user_name'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### All setup ,goto: http://public_ip and apache will server required packages