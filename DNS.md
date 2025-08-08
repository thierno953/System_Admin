# DNS Server with BIND9 on Ubuntu Server

### Static IP Configuration with Netplan

```sh
sudo nano /etc/netplan/00-installer-config.yaml
```

- Content

```sh
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: false
    enp0s8:
      addresses: [192.168.129.172/24]
      nameservers:
        addresses: [1.1.1.1, 8.8.8.8]
```

### Apply the Configuration

```sh
sudo netplan apply
```

### Install BIND9 and Enable Firewall

```sh
sudo apt update
sudo apt install bind9 bind9-utils -y
```

### Enable and Allow UFW (`Firewall`)

```sh
sudo ufw enable
sudo ufw allow bind9
sudo ufw status
```

### Start BIND9

```sh
sudo systemctl start bind9
sudo systemctl status bind9
```

### Configure BIND: `named.conf.options`

```sh
sudo nano /etc/bind/named.conf.options
```

```sh
options {
    directory "/var/cache/bind";
    listen-on { any; };
    allow-query { localhost; 192.168.129.0/24; };
    forwarders {
        8.8.8.8;
    };
    dnssec-validation no;
};
```

### Force IPv4 Usage

```sh
sudo nano /etc/default/named
```

```sh
OPTIONS="-u bind -4"
```

```sh
sudo named-checkconf
sudo systemctl restart bind9
```

### Declare DNS Zones

```sh
sudo nano /etc/bind/named.conf.local
```

- Content

```sh
zone "diarabaka.local" IN {
    type master;
    file "/etc/bind/zones/db.diarabaka.local";
};

zone "129.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.129.168.192";
};
```

### Create Zone Files

```sh
sudo mkdir -p /etc/bind/zones
sudo cp /etc/bind/db.local /etc/bind/zones/db.diarabaka.local
```

### Forward Zone: `db.diarabaka.local`

```sh
sudo nano /etc/bind/zones/db.diarabaka.local
```

```sh
$TTL 604800
@       IN  SOA dns01.diarabaka.local. root.diarabaka.local. (
                2         ; Serial
           604800         ; Refresh
            86400         ; Retry
          2419200         ; Expire
           604800 )       ; Negative Cache TTL
;
        IN  NS  dns01.diarabaka.local.
dns01   IN  A   192.168.129.172
server  IN  CNAME dns01
```

### Reverse Zone: `db.129.168.192`

```sh
sudo cp /etc/bind/zones/db.diarabaka.local /etc/bind/zones/db.129.168.192
sudo nano /etc/bind/zones/db.129.168.192
```

```sh
$TTL 604800
@       IN  SOA dns01.diarabaka.local. root.diarabaka.local. (
                2         ; Serial
           604800         ; Refresh
            86400         ; Retry
          2419200         ; Expire
           604800 )       ; Negative Cache TTL
;
        IN  NS  dns01.diarabaka.local.
172     IN  PTR dns01.diarabaka.local.
```

### Configuration Check

```sh
sudo named-checkconf /etc/bind/named.conf.local
sudo named-checkzone diarabaka.local /etc/bind/zones/db.diarabaka.local
sudo named-checkzone 129.168.192.in-addr.arpa /etc/bind/zones/db.129.168.192
```

### Restart BIND9

```sh
sudo systemctl restart bind9
sudo systemctl status bind9
```

### Local DNS Configuration (Resolution)

```sh
sudo nano /etc/resolv.conf
```

- Exemple

```sh
nameserver 192.168.129.172
nameserver 127.0.0.53
options edns0 trust-ad
search diarabaka.local
```

### Prevent `resolv.conf` from Being Overwritten

```sh
sudo cp /etc/resolv.conf /etc/resolv.conf.back
sudo chattr +i /etc/resolv.conf.back
sudo rm -f /etc/resolv.conf
sudo cp /etc/resolv.conf.back /etc/resolv.conf
```

### DNS Resolution Tests

```sh
host dns01.diarabaka.local
ping dns01.diarabaka.local
host dns01
host 192.168.129.172
```

### Securing the Setup

- Verify permissions:

```sh
sudo chown -R bind:bind /etc/bind/zones
```
