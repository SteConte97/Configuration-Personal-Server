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





























