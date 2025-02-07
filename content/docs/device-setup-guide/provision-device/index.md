+++
title = 'Provision Device'
date = 2024-09-24T16:18:54-07:00
draft = false
weight = 4
+++
These steps are to bootstrap the device into something more useful and easy to connect to. We will start by connecting with a physical cable over adb, but end with connecting via ssh. In the process, we will also fix a few items that are broken in the base image.

## Connect with ADB
We will first connect to the device using ADB on the host over the USB-C cable.
### Install ADB on the host
```sh
sudo apt install adb
```
### Connect

![USB-C Port](usb-c-port.png "USB-C Port")

1. Plug the device into the host computer using the USB-C cable.

2. Run the following command from the host to enter a root shell on the device.
```sh
adb wait-for-device && adb root && adb shell
# you will be left at a shell prompt:
# sh-5.0#
```

## Connect to WiFi
As a starting point, we want to connect the device to the internet to download some new packages. From the device, run:
```sh
wpa_cli # you will now be at the wpa_cli shell prompt
remove_network 0
add_network
set_network 0 ssid "YOUR NETWORK SSID"
set_network 0 psk "YOUR NETWORK PSK"
enable_network 0
quit
```

## Fix apt
```sh
apt update
apt install libvulkan1=1.2.131.2-1
apt install -o DPkg::options::="--force-overwrite" libc6-arm64-cross
apt update
apt upgrade
apt autoremove
```

## Set up better network management
The RB5 image defaults to using `wpa_supplicant`, `dhcpcd`, and `dnsmasq` to configure networking. Much more modern alternatives to these programs exist now, so we can use those for an overall better experience. In this case, we'll set up `NetworkManager` and `systemd-resolved`. There are a few hacks required to make it work because of some quirks in the image.
```sh
apt install network-manager
```
```sh
systemctl disable dhcpcd
```
```sh
rm /data/misc/wifi/wpa_supplicant.conf
```
```sh
# The default resolved.conf file disables the stub listener. Ordinarily, we
# could just edit the file to enable it, but the changes don't persist through
# reboots for whatever reason. Instead, if we delete the file, systemd-resolved
# uses the default values, which enables the stub listener.
rm /etc/systemd/resolved.conf
```
```sh
apt remove dnsmasq
```
```sh
# reboot the device and connect again via adb
exit # to return to the host system
adb reboot
```
{{< callout type="warning" >}}
  Do not be tempted to reboot by running `reboot` from the device. For whatever reason, this tends to be inconsistent. Using ADB is more reliable.
{{< /callout >}}
```sh
# after rebooting, NetworkManager should be active, meaning that we can use
# nmtui to set up WiFi
nmtui
```
```sh
# alternatively, you can connect directly from the command line, but beware 
# that your password may end up in a .bash_history file.
nmcli device wifi connect "SSID" password "password"
```
```sh
# set the connection into "stable" MAC mode so that DHCP doesn't keep assigning
# new addresses.
# this step is optional but highly recommended
nmcli con modify "SSID" wifi.cloned-mac-address stable
```

## Set up mDNS
mDNS allows hosts on the same local network to discover each other more easily. On Linux, `avahi` is the most common mDNS implementation.

1. Set a hostname on the device
```sh
# replace my-rb5 with whatever you would like. Eventually, you will use this
# name to discover the device.
hostnamectl set-hostname my-rb5
```
2. Install `avahi`
```sh
apt install avahi-daemon avahi-utils
```
3. Test from the host machine
{{< callout type="warning" >}}
  Out of the box, an Ubuntu 24.04 system with systemd-resolved does not enable
  mDNS. You can fix this by editing `/etc/systemd/resolved.conf` to contain the
  line `MulticastDNS=yes`, then restarting systemd-resolved. Other systems may
  or may not require this extra configuration.
{{< /callout >}}
```sh
# replace my-rb5 with whatever name you chose earlier
resolvectl query my-rb5.local
# you should see output with an IP address that matches the address of the device
```
```sh
# additionally, check that you can ping the device
ping my-rb5.local
```

## Set up ssh
{{< callout type="warning" >}}
  These are quick-and-dirty instructions to get into the system that **do not** follow best security practice. You'll definitely want to harden or disable ssh access if you're deploying the system in production.
{{< /callout >}}
1. Allow the root user to log in with a password
```sh
# on the device
echo "PermitRootLogin yes" >/etc/ssh/sshd_config.d/permitrootlogin.conf
```
2. Restart sshd for the changes to take effect
```sh
# on the device
systemctl restart ssh
```
3. Connect from the host
```sh
ssh root@my-rb5.local
# on first connection, it will ask you whether you trust the ssh key fingerprint
# of the device -- in this case, we do
# default password is "oelinux123"
```
