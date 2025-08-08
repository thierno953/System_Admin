# Web Server with Apache on Ubuntu Server

### Installing Apache

```sh
sudo apt install apache2 -y
```

### Checking Apache Status

```sh
sudo systemctl status apache2
```

### Creating the Directory for the Website

```sh
sudo mkdir /var/www/medium.diarabaka.local
sudo chown -R $USER:$USER /var/www/medium.diarabaka.local
sudo chmod -R 755 /var/www/medium.diarabaka.local
```

### Sample Web Page

```sh
sudo nano /var/www/medium.diarabaka.local/index.html
```

- Content

```sh
<h1>Hello Apache2 PROX</h1>
```

### Virtual Host Configuration File

```sh
sudo nano /etc/apache2/sites-available/medium.diarabaka.local.conf
```

- Initial Content:

```sh
<VirtualHost *:80>
    ServerAdmin medium@diarabaka.local
    ServerName medium.diarabaka.local
    ServerAlias www.medium.diarabaka.local
    DocumentRoot /var/www/medium.diarabaka.local
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

### Enabling the Site Configuration

```sh
sudo a2ensite medium.diarabaka.local.conf
sudo a2dissite 000-default.conf
sudo apache2ctl configtest
sudo systemctl restart apache2
```

### Hosts File (on the local machine or server)

```sh
sudo nano /etc/hosts
```

- Add

```sh
192.168.129.172 medium.diarabaka.local
```

### Local DNS Zone with BIND9

```sh
sudo nano /etc/bind/zones/db.diarabaka.local
```

- Content

```sh
medium.diarabaka.local  IN  A  192.168.129.172
```

```sh
sudo systemctl restart bind9
sudo systemctl status bind9
```

## Enable HTTPS with a Self-Signed Certificate

### Enable SSL Module

```sh
sudo a2enmod ssl
sudo systemctl restart apache2
```

### Generate SSL Certificate

```sh
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
-keyout /etc/ssl/private/apache-selfsigned.key \
-out /etc/ssl/certs/apache-selfsigned.crt
```

### Update the Virtual Host File for HTTPS

```sh
sudo nano /etc/apache2/sites-available/medium.diarabaka.local.conf
```

- New content:

```sh
<VirtualHost *:80>
    ServerName medium.diarabaka.local
    Redirect / https://medium.diarabaka.local/
</VirtualHost>

<VirtualHost *:443>
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
    SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key

    ServerAdmin medium@diarabakacomputer.local
    ServerName medium.diarabaka.local
    ServerAlias www.medium.diarabaka.local
    DocumentRoot /var/www/medium.diarabaka.local
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

### Test and Reload Apache

```sh
sudo apache2ctl configtest
sudo systemctl reload apache2
sudo a2dissite 000-default
```

- `https://medium.diarabaka.local` is up and running with a secure (self-signed) HTTPS certificate.
