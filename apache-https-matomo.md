## After server on Http, now moving to Https as:

### Using certbot for ssl certificate:
```
sudo apt update
sudo apt install certbot python3-certbot-apache
```
- Obtain an SSL certificate using xip.io (a wildcard DNS service).
` sudo certbot --apache -d public_ip.xip.io`
- Follow the prompts to complete the certificate installation.

- Certbot will automatically configure Apache to use the SSL certificate. However, we can check the Apache configuration file to verify:
```
sudo nano /etc/apache2/sites-available/000-default-le-ssl.conf
```
- Ensure the configuration includes SSL settings similar to this:
```
<IfModule mod_ssl.c>
    <VirtualHost *:443>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        ServerName public_ip.xip.io
        SSLCertificateFile /etc/letsencrypt/live/public_ip.xip.io/fullchain.pem
        SSLCertificateKeyFile /etc/letsencrypt/live/public_ip.xip.io/privkey.pem
        Include /etc/letsencrypt/options-ssl-apache.conf
    </VirtualHost>
</IfModule>
```
- Enable the SSL module and the new site configuration
```
sudo a2enmod ssl
sudo a2ensite 000-default-le-ssl
sudo systemctl reload apache2
```
- Ensure Apache is running and check for any configuration errors:
```
sudo systemctl status apache2
sudo apache2ctl configtest
```
Access : https://20.197.55.76.xip.io

# If certbot fails, we can self-generate certificates using OPENSSL:
- Access vm
- Generate a self-signed SSL certificate
```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt
```
```
sudo nano /etc/apache2/sites-available/default-ssl.conf
```
- Paste the following configuration, adjusting paths and domains as necessary:
```
<IfModule mod_ssl.c>
<VirtualHost _default_:443>
    ServerAdmin webmaster@localhost
    ServerName public_ip

    DocumentRoot /var/www/html

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    SSLEngine on

    SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
    SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key

    <FilesMatch "\.(cgi|shtml|phtml|php)$">
        SSLOptions +StdEnvVars
    </FilesMatch>

    <Directory /usr/lib/cgi-bin>
        SSLOptions +StdEnvVars
    </Directory>

    BrowserMatch "MSIE [2-6]" \
        nokeepalive ssl-unclean-shutdown \
        downgrade-1.0 force-response-1.0
    BrowserMatch "MSIE [17-9]" ssl-unclean-shutdown

</VirtualHost>
</IfModule>

```
- Enable the SSL module, enable the default SSL virtual host, and restart Apache:
```
sudo a2enmod ssl
sudo a2ensite default-ssl
sudo systemctl restart apache2
```


### Other commands:
- Review Apache error logs ` sudo tail -n 50 /var/log/apache2/error.log`
- Use `sudo apache2ctl configtest` to check for syntax errors:
- Check status: `sudo systemctl status apache2.service`
- Restart apache server: `sudo systemctl start apache2`
- Verify Port Availability `sudo ss -tlnp | grep :80` and `sudo ss -tlnp | grep :443 ` or `sudo ss -tlnp | grep ':80\|:443'` 
- Free port using `sudo kill -9 PID_ID`
- check the system-wide logs using journalctl for any system-level issues that might affect Apache: `sudo journalctl -xe`

## IF default apache page is server instead of required package : 
- Go to config files(`/etc/apache2/sites-available` and `ls`) and make sure DocumentRoot is set to `/var/www/html/matomo` to make it serve default