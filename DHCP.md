# Installation and Configuration of a DHCP Server on Linux

### Install Required Packages

```sh
sudo apt update
sudo apt install isc-dhcp-server iptables-persistent vim -y
```

### Configure Network Interface

```sh
sudo vi /etc/netplan/01-static-ip.yaml
```

```sh
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      addresses:
        - 192.168.10.2/24
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
      routes:
        - to: default
          via: 192.168.10.254
```

```sh
sudo netplan apply
```

### Define the DHCP Interface

```sh
sudo cp /etc/default/isc-dhcp-server /etc/default/isc-dhcp-server.bkp
echo 'INTERFACESv4="enp0s3"' | sudo tee /etc/default/isc-dhcp-server
```

### Configure the DHCP Server

- Backup and edit the main config file:

```sh
sudo cp /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.bkp
sudo vim /etc/dhcp/dhcpd.conf
```

- Clean out comments:

```sh
:g/^\s*#/d
```

- Add the following configuration:

```sh
# Global settings
option domain-name "example.local";
option domain-name-servers 8.8.8.8, 1.1.1.1;
default-lease-time 600;
max-lease-time 7200;
ddns-update-style none;
authoritative;

# Main subnet
subnet 192.168.10.0 netmask 255.255.255.0 {
    range 192.168.10.50 192.168.10.200;
    option routers 192.168.10.254;
    option subnet-mask 255.255.255.0;
    option domain-name-servers 8.8.8.8, 1.1.1.1;
}

# Static reservation (example: printer)
host reserved-client {
    hardware ethernet 08:00:27:bd:22:0c;
    fixed-address 192.168.10.50;
}
```

#### Explanation

- `range` -> Pool of dynamically assigned IP addresses
- `host reserved-client` -> Static IP for a device (e.g., printer)
- `default-lease-time` -> Lease duration before renewal

### Start and Verify the DHCP Server

```sh
sudo dhcpd -t                             # Syntax check
sudo systemctl restart isc-dhcp-server    # Restart the service
sudo systemctl enable isc-dhcp-server     # Enable on boot
sudo systemctl status isc-dhcp-server     # Check status
```

### Enable NAT for Internet Access

```sh
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

- Configure NAT rules (replace enp0s8 with your internet-facing interface):

```sh
sudo iptables -t nat -A POSTROUTING -o enp0s8 -j MASQUERADE
sudo iptables -A FORWARD -i enp0s3 -o enp0s8 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -j ACCEPT
sudo iptables-save | sudo tee /etc/iptables/rules.v4
```

### Lease Verification and Monitoring

```sh
sudo dhcp-lease-list                          # View active leases
tail -f /var/log/syslog | grep dhcpd          # Live DHCP logs
journalctl -xeu isc-dhcp-server.service       # System logs for the service
```

### Client-Side Troubleshooting (Linux)

```sh
sudo dhclient -v        # Verbose IP request
ip a                    # Check received address
```
