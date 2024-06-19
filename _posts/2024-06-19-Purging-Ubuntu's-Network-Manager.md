---
title: "Purging Ubuntu's Network Manager"
date: 2024-06-19 13:30 +530
categories: [tutorial] # thoughts can be another category
tags: [networking, linux, networkmanager, purge] # make sure its always lower case
---

*For latest updates and download all recent relevant files, please refer GH repository [here](https://github.com/primeDevansh/purgeNetworkManager/tree/main).*

# Purge Network Manager and Hardcode Network Configurations

> Intended for Ubuntu Desktop 20.04 LTS and later versions.

> Implemented on Ubuntu Desktop 20.04 LTS (Focal Fossa).

When you want to purge the Network Manager on Ubuntu 20.04 and hardcode the network configuration, you should follow these steps in order:

1. Hardcode the network configuration: Before removing the Network Manager, it's crucial to ensure that you have a static network configuration in place. This will ensure you maintain network connectivity after the Network Manager is removed.

2. Purge the Network Manager: Once the static network configuration is set up and verified to be working, you can proceed to remove the Network Manager.

3. Verifying Network Configurations After a Reboot.

We will be using **Netplan** to setup (hardcode) network interfaces.

Netplan is a utility developed by Canonical, the company behind Ubuntu. It is used to setup network interfaces on recent versions of Ubuntu. It provides a network configuration abstraction over the currently supported two “backend” system (“renderer” in Netplan terminology): networkd and NetworkManager. Using Netplan, both physical and virtual network interfaces are configured via yaml files which are translated to configurations compatible with the selected backend. ([Netplan network configuration tutorial for beginners](https://linuxconfig.org/netplan-network-configuration-tutorial-for-beginners))

## Detailed Steps

### Step 1: Hardcoding the Network Configuration

1. Identify your network interface: Use the 
    ```bash
    ip a
    ```
    or 
    ```bash
    ifconfig
    ```
    command to identify your network interface name (e.g., eth0, enp0s3, etc.).
    *In our case, we configured enp4s0 network interface.*

    > **Note down ALL the configurations (IP addr, subnet, router, gateway) associated with your target interface. They will be used later when we'd hardcode the network configuration.**

2. Edit the /etc/netplan/01-netcfg.yaml file (or create it if it does not exist):

    ```bash
    sudo nano /etc/netplan/01-netcfg.yaml
    ```

3. Add your static network configuration: **Replace** enp4s0 with your actual network interface name and configure the IP address, gateway, and DNS servers as needed. 

    > The IP address (192.168.22.130) also contains the subnet (/20) associated with it. Make sure to add it properly as shown below. **Also make sure to follow proper indentation as well because we are dealing with a .yaml file**

    Here's the configuration file we used: [01-netcfg.yaml](/assets/posts_assets/2024-06-19-Purging-Ubuntu's-Network-Manager/01-netcfg.yaml)

    ```yaml
    network:
      version: 2
      ethernets:
        enp4s0:                 # REPLACE enp4s0 with the intended interface name
          dhcp4: no             # Disable DHCP for IPv4
          addresses:
            - 192.168.22.130/20     # The static IP address and subnet mask
          gateway4: 192.168.16.11   # Your gateway (router) IP address
          nameservers:
            addresses:
              - 8.8.8.8        # Primary DNS server (Google DNS)
              - 8.8.4.4        # Secondary DNS server (Google DNS)
    ```

4. Disable the existing netplan file: You can disable the 01-network-manager-all.yaml file by renaming it, which effectively prevents it from being applied.

    ```bash
    sudo mv /etc/netplan/01-network-manager-all.yaml /etc/netplan/01-network-manager-all.yaml.bak
    ```

5. Apply the configurations

    ```bash
    sudo netplan apply
    ```

6. Verify the network configuration: Ensure you can still connect to the network. You can use ping, curl, or any other network tool to verify the connectivity.

    - Make sure to test the internet connectivity.
    - Make sure to test local access through ssh.

### Step 2: Purging the Network Manager

The steps to purge network manager depends on the desktop environment. Once the static network configuration is **confirmed** to be working, you can safely remove the Network Manager. 

1. For Ubuntu MATE 18.04 LTS and 20.04 LTS purging network-manager package is safe. You can simply run:

    ```bash
    sudo apt-get purge network-manager
    ```

2. For Ubuntu 18.04 LTS and 20.04 LTS with GNOME desktop (**our case**) purging network-manager package will also purge ubuntu-desktop and gnome-control-center (essential part of GNOME desktop). So it is not an option.

    Here you should stop NetworkManager and some other services:

    ```bash
    sudo systemctl stop NetworkManager.service
    sudo systemctl stop NetworkManager-wait-online.service
    sudo systemctl stop NetworkManager-dispatcher.service
    sudo systemctl stop network-manager.service
    ```

    **Check for the status of Network Manager if it is still running. This can be done in either of the 2 ways described below:**

    - Open System Settings > Network

        If it shows an error opening network settings, network manager has been successfully disabled.

    OR 

    - Through command line

        ```bash
        nmcli device
        ```

        ![NM is not running](/assets/posts_assets/2024-06-19-Purging-Ubuntu's-Network-Manager/nmcli_device.png)

    Now, disable network manager (permanently) to avoid it restarting after a reboot:

    ```bash
    sudo systemctl disable NetworkManager.service
    sudo systemctl disable NetworkManager-wait-online.service
    sudo systemctl disable NetworkManager-dispatcher.service
    sudo systemctl disable network-manager.service
    ```

Referenced from:

([How do I disable network manager permanently?](https://askubuntu.com/questions/1091653/how-do-i-disable-network-manager-permanently))

([Stopping and Disabling NetworkManager](https://help.ubuntu.com/community/NetworkManager#Stopping_and_Disabling_NetworkManager))

### Step 3: Rebooting the System & Final Verification

1. Reboot your system: Make sure to reboot your system into the **same** OS.

    ```bash
    sudo reboot
    ```

2. **Re-verify the network configuration**: Make sure to re-verify the network configuration once again as done previously as well. Ensure you can still connect to the network. You can use ping, curl, or any other network tool to verify the connectivity.

    - Make sure to test the internet connectivity.
    - Make sure to test local access through ssh.

3. Re-verify Network Manager's status: Check if Network Manager is running or not. This can be done following the steps as described before.

> Congratulations! If you've made it till here, you've successfully purged Network Manager and hardcoded the network configurations.