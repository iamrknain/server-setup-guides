# Table of Contents:
1. [Install Apache](#install-apache)
2. [Enable Modules For Proxying](#enable-modules-for-proxying)
3. [Configure Apache Virtual Host](#configure-apache-virtual-host)
3. [Configure Apache Port](#configure-apache-port)
4. [Apache Server Commands](#apache-server-commands)


## Install Apache:
```bash
sudo apt update
sudo apt install apache2
```

# Enable Modules For Proxying
```bash
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo a2enmod proxy_balancer
sudo a2enmod lbmethod_byrequests
```

# Configure Apache Virtual Host
```bash
sudo nano /etc/apache2/sites-available/new_virtal_host.conf
```
```apache
# Add the following configuration to proxy requests to server for required port

<VirtualHost *:80>
    # Admin email, where server problems will be sent
    ServerAdmin admin@gmail.com

    # The primary domain name/ public_ip for this virtual host (eg.: 20.40.47.132 )
    ServerName domain.com

    # Alternative names for this virtual host
    ServerAlias www.domain.com

    # The location of the root directory for this virtual host (eg: /var/www/new_src)
    DocumentRoot /var/www/example.com/public_html/ 

    # Preserve the original host header
    ProxyPreserveHost On

    # Proxy requests to the backend server
    ProxyPass / http://20.40.47.132:8080/
    ProxyPassReverse / http://20.40.47.132:8080/

    # Location of the error log file
    ErrorLog ${APACHE_LOG_DIR}/example.com_error.log

    # Location of the access log file and the log format
    CustomLog ${APACHE_LOG_DIR}/example.com_access.log combined

    # Directory settings for the document root
    <Directory /var/www/example.com/public_html>
        # Allow directory listings if no index file is present
        Options Indexes FollowSymLinks

        # Allow .htaccess files to override Apache's settings
        AllowOverride All

        # Allow all access to this directory
        Require all granted
    </Directory>
</VirtualHost>
```

```bash
# Enable Necessary Modules

sudo a2enmod proxy
sudo a2enmod proxy_http

# Enable the new virtual host configuration:

sudo a2ensite new_virtal_host.conf

# Set proper permissions for new root/src:

sudo chown -R $USER:$USER /var/www/new_src
sudo chmod -R 755 /var/www

sudo systemctl restart apache2 # start apache server
```

# Configure Apache Port
```bash
sudo nano /etc/apache2/ports.conf
```

```apache
# Listen on all IP addresses (IPv4 and IPv6) for HTTP traffic on port 80
Listen 0.0.0.0:80
Listen [::]:80

# Listen on an additional port 8080 for HTTP traffic
Listen 8080

# Listen on all IP addresses (IPv4 and IPv6) for HTTPS traffic on port 443
<IfModule ssl_module>
    Listen 0.0.0.0:443
    Listen [::]:443
</IfModule>

<IfModule mod_gnutls.c>
    Listen 0.0.0.0:443
    Listen [::]:443
</IfModule>

# Listen on an additional HTTPS port 8443
Listen 8443

# Also modify/ update a corresponding virtaul host 

# Expample for Port 8443
```
```apache
<VirtualHost *:8443>
    # Admin email, where server problems will be sent
    ServerAdmin admin@example.com

    # The IP address for this virtual host
    ServerName 20.40.47.132

    # The location of the root directory for this virtual host
    DocumentRoot /var/www/example.com/public_html

    # Location of the error log file
    ErrorLog ${APACHE_LOG_DIR}/20.40.47.132_error.log

    # Location of the access log file and the log format
    CustomLog ${APACHE_LOG_DIR}/20.40.47.132_access.log combined

    # Enable SSL for this virtual host
    SSLEngine on

    # Path to the SSL certificate file
    SSLCertificateFile /path/to/your/certificate.crt

    # Path to the SSL certificate key file
    SSLCertificateKeyFile /path/to/your/private.key

    # Directory settings for the document root
    <Directory /var/www/example.com/public_html>
        # Allow directory listings if no index file is present
        Options Indexes FollowSymLinks

        # Allow .htaccess files to override Apache's settings
        AllowOverride All

        # Allow all access to this directory
        Require all granted
    </Directory>
</VirtualHost>

```

```bash
# enable necessary Apache modules

sudo a2enmod proxy
sudo a2enmod proxy_http
sudo a2enmod ssl
sudo a2enmod gnutls
```


# Apache Server Commands:
```bash
sudo systemctl restart apache2 # start server
sudo apachectl configtest # check sytax
sudo tail -n 50 /var/log/apache2/error.log # apache logs

iisreset /stop # If IIS is running, stop it to start apache server
udo netstat -tuln | grep :80 # Check Active Ports

# To make server’s firewall allows incoming connections on port eg. 80. We can use ufw or iptables to check and modify firewall settings:

sudo ufw allow 80/tcp
sudo ufw reload

```
