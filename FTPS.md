# FTPS Server Configuration

### Update and Install

```sh
sudo apt update && sudo apt install vsftpd -y
```

### Backup the Original Configuration

```sh
sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.original
```

### Configure vsftpd

```sh
sudo nano /etc/vsftpd.conf
```

- Add or modify the following lines:

```sh
# Network
listen=YES
listen_ipv6=NO

# Authentication
anonymous_enable=NO
local_enable=YES
write_enable=YES

# Security and user isolation
chroot_local_user=YES
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd.chroot_list
allow_writeable_chroot=YES
secure_chroot_dir=/var/run/vsftpd/empty

# Passive mode
pasv_enable=YES
pasv_min_port=40000
pasv_max_port=50000
pasv_address=192.168.129.20  # Replace with your server’s IP address

# SSL/TLS
rsa_cert_file=/etc/ssl/certs/vsftpd.pem
rsa_private_key_file=/etc/ssl/private/vsftpd.pem
ssl_enable=YES
allow_anon_ssl=NO
force_local_data_ssl=YES
force_local_logins_ssl=YES
ssl_tlsv1=YES
ssl_sslv2=NO
ssl_sslv3=NO
require_ssl_reuse=NO
ssl_ciphers=HIGH
```

### Create the FTP User

```sh
sudo useradd -m -d /home/thierno -s /bin/bash thierno
sudo passwd thierno  # Set the password
```

### Allow the User in chroot

```sh
echo "thierno chroot" | sudo tee /etc/vsftpd.chroot_list
```

### Generate SSL Certificate

```sh
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/private/vsftpd.pem \
  -out /etc/ssl/certs/vsftpd.pem \
  -subj "/C=BE/ST=BXL/L=BXL/O=MyOrg/CN=ftp-server"
```

### Configure UFW Firewall

```sh
sudo ufw allow 20/tcp           # FTP Data
sudo ufw allow 21/tcp           # FTP Control
sudo ufw allow 990/tcp          # Implicit FTPS
sudo ufw allow 40000:50000/tcp  # Passive mode range
sudo ufw enable
sudo ufw status
```

### Restart and Enable vsftpd

```sh
sudo systemctl restart vsftpd
sudo systemctl enable vsftpd
sudo systemctl status vsftpd
```

### Secure the User’s Home Directory

```sh
sudo chmod 750 /home/thierno
sudo mkdir -p /home/thierno/ftp
sudo chown -R thierno:thierno /home/thierno/ftp
```

### Test FTPS Connection

```sh
curl -k --ssl-reqd ftp://localhost/ -u thierno
```

### With password in the command (not recommended for security reasons):

```sh
curl -k --ssl-reqd ftp://localhost/ -u thierno:mypassword
```
