# Setting Up a Secure Samba Share with User Management

### Install Samba

```sh
sudo apt update
sudo apt install samba -y
samba --version
```

### Create a User and Prepare Directories

- Create user **john**, the group **smbgroup**, and the shared folder

```sh
sudo adduser john
sudo groupadd smbgroup
sudo mkdir /home/group
sudo chgrp smbgroup /home/group
sudo chmod 770 /home/group
sudo chmod g+s /home/group
```

### Backup and Edit the Samba Configuration File
 
- Backup the Samba config file and open it for editing:

```sh
cd /etc/samba/
sudo cp smb.conf smb.conf.bak
sudo nano smb.conf
```

- Lines to add or modify in **smb.conf**:

```sh
[global]
workgroup = WORKGROUP
security = user
interfaces = lo enp0s3
bind interfaces only = yes

[homes]
comment = Personal folders
path = /home/%S
valid users = %S
read only = no
create mask = 0700
directory mask = 0700
browseable = no

[Group]
comment = Shared folder for smbgroup members
path = /home/group
writable = yes
guest ok = no
valid users = @smbgroup
force create mode = 0770
force directory mode = 0770
inherit permissions = yes
```

### Set Access Permissions

- Set permissions and add user **john** to group **smbgroup**:

```sh
sudo chmod 755 /home/john
sudo smbpasswd -a john
sudo usermod -aG smbgroup john
```

### Verify Samba Configuration

```sh
testparm
```

### Restart Samba Service and Open Firewall

```sh
sudo systemctl restart smbd
sudo systemctl status smbd
sudo ufw enable
sudo ufw allow samba
```

### Test Connection with smbclient

- Test access using **smbclient** (from the local machine):

```sh
sudo apt install smbclient
smbclient //192.168.129.152/Group -U john
```

- Once connected:

```sh
smb: \> mkdir testfolder
smb: \> put /etc/hosts hosts.txt
smb: \> dir
```

### Recommended Next Test

- Connect from a `Windows workstation`:

- Open File Explorer and enter `\\192.168.129.152\Group`, then log in with user `john`.

### Add Another User Alice and Add to smbgroup

```sh
sudo adduser alice --home /home/alice
sudo smbpasswd -a alice
sudo usermod -aG smbgroup alice
```

- **Test with a user who is not a group member** to verify access control:

  - Should return an access denied error (NT_STATUS_ACCESS_DENIED)

```sh
sudo adduser bob
sudo smbpasswd -a bob
smbclient //192.168.129.152/Group -U bob
```
