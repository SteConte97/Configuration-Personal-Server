# Installation guide for a personal server

## Introduction

This document is a comprehensive guide to setting up your own development server. Whether you are a technical expert or just a geek, this guide will walk you through the process of installing essential software tools such as .NET and PostgreSQL on an Ubuntu server. By the end of this guide, you will have a fully functional development workspace.

## Description

In this guide, you will find step-by-step instructions for installing the following components:

- **Ubuntu**: A popular Linux distribution that is user-friendly and highly customizable.
- **.NET SDK**: A platform for building and running applications on various operating systems.
- **PostgreSQL**: A powerful open-source relational database management system known for its reliability and performance.
- **Additional Tools**: Any other necessary tools and libraries to enhance your development experience (such as nginx).

## Prerequisites

Before we begin with the installations, ensure that you have:

- A compatible computer or virtual environment to run Ubuntu.
- An active internet connection for downloading packages.

## Steps

Ubuntu Server LTS is available [here](https://www.ubuntu-it.org/download), once you have downloaded the ISO file proceed with the installation on a physical machine or a virtual machine following the ubuntu wizard.

Once ubuntu has been installed, proceed with the package update:
```
sudo apt update
sudo apt upgrade
```

### .NET SDK

Proceed with the [installation](https://learn.microsoft.com/en-us/dotnet/core/install/linux-ubuntu-install?tabs=dotnet8&pivots=os-linux-ubuntu-2410) of .NET SKD 8.0:
```
sudo apt-get install -y dotnet-sdk-8.0
```

To verify the correct installation run the following command and check if it gives all the information about the installed version of .NET:
```
dotnet --info
```

In this repository I have uploaded an MVC test project already configured.

### Static IP

First you need to check the active network intefrastructure (e.g. eth0, enp0s3, etc...):
```
ip link
```

Start the ubuntu text editor and configure the ip static:
```
sudo nano /etc/netplan/00-installer-config.yaml
```

Paste this configuration:
```
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:  # Replace with the name of your interface
      dhcp4: false
      addresses:
        - 192.168.1.100/24  #  Static IP address and subnet mask
      gateway4: 192.168.1.1  # Gateway
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4  # DNS server
```

Apply the changes:
```
sudo netplan apply
```

### NGINX

Proceed with the installation of nginx as a reverse proxy:
```
sudo apt-get install nginx
```

Add file configuration:
```
sudo nano /etc/nginx/sites-available/app
```

Add the following configuration:
```
server {
    listen 80;
    server_name your-domain.com; # Or your public IP if you don't have a domain

    location / {
        proxy_pass http://localhost:5000;  # Redirects to the ASP.NET app
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection keep-alive;
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Activates the new configuration and deletes the default configuration:
```
sudo ln -s /etc/nginx/sites-available/app /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl restart nginx
```


Create the service connected to the .NET app:
```
nano /etc/systemd/system/[YourAppName].service
```

Add the following configuration (replace "YourAppName"):
```
[Unit]
Description=YourAppName

[Service]
WorkingDirectory=/var/www/app/
ExecStart=/usr/bin/dotnet /var/www/app/YourAppName.dll
Restart=always
# Restart service after 10 seconds if the dotnet service crashes:
RestartSec=10
KillSignal=SIGINT
SyslogIdentifier=dotnet-YourAppName
User=root
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false

[Install]
WantedBy=multi-user.target
sudo systemctl enable [YourAppName].service
sudo systemctl start [YourAppName].service
```





























