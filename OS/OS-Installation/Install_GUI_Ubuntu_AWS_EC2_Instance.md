
# How to Install GUI on Ubuntu AWS EC2 Instance

This guide will walk you through the process of setting up a Graphical User Interface (GUI) on an Ubuntu AWS EC2 instance. The steps are straightforward 

## Prerequisites
- An Ubuntu AWS EC2 instance
- SSH access to the instance

## Steps

### Switch to the root user

```sh
sudo su -
```

### Update and upgrade your system packages

```sh
apt update -y
apt upgrade -y
```

### Enable password authentication for SSH

Modify the SSH configuration file to allow password authentication.

```sh
sed -i 's/^PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
/etc/init.d/ssh restart
```

### Create a new user

Replace `username` with the user name of your choice and set a password when prompted.

```sh
adduser username
```

### Grant sudo privileges to the new user

Edit the sudoers file to give the new user root privileges.

```sh
visudo
```

Add the following line to the file:

```sh
username ALL=(ALL:ALL) ALL
```

Save and exit the file.

### Install the GUI packages

Install the necessary packages for the GUI and remote desktop protocol.

```sh
apt install xrdp xfce4 xfce4-goodies tightvncserver -y
```

### Set a hostname

Replace `hostgui` with the hostname of your choice.

```sh
hostnamectl set-hostname hostgui
systemctl restart systemd-hostnamed
```

### Logout and log back in as the new user

```sh
su - username
```

### Configure the session for the new user

```sh
echo xfce4-session > ~/.xsession
sudo cp ~/.xsession /etc/skel
sudo sed -i '0,/-1/s//ask-1/' /etc/xrdp/xrdp.ini
sudo service xrdp restart
sudo shutdown -r now
```

### Log back in

After the reboot, log back in as the new user.

### Install PuTTY (optional)

If you prefer using PuTTY for SSH access, you can install it.

```sh
sudo su -
apt-get install putty -y
```

### Connect to the instance via RDP

On your local machine, press `Windows + R`, type `mstsc`, and hit Enter. Enter the public IP address of your instance, and then log in using the username and password you created.

## Summary
You've successfully installed and configured a GUI on your Ubuntu AWS EC2 instance. You can now connect to your instance using Remote Desktop Protocol (RDP) and enjoy a graphical interface.


