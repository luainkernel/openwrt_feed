<!-- markdownlint-disable MD013 -->
# OpenWrt config for Raspberry Pi 3

> [!NOTE]
> The configuration for [Raspberry Pi 3](https://openwrt.org/toh/hwdata/raspberry_pi_foundation/raspberry_pi_3_b) was derived from [Vinicius Paes' post1](https://viniciuspaes.com/raspberry-pi/how-to-build-custom-openwrt-image-raspberry-pi-router/) and [Cleiton Bueno's post](https://embarcados.com.br/openwrt-customizando-e-embarcando-na-raspberry-pi/#RaspberryPI-3B3B3CM-e-RaspberryPI)

## Overview

### Target configuration

```txt
Target System --->
    (X) Broadcom BCM27xx
    Subtarget --->
    (X) BCM2710 boards (64 bit)
Target Profile --->
    (X) Raspberry Pi 2B-1.2 (64bit) (Raspberry Pi 3B/3B+/3CM (64bit)
    (X) Raspberry Pi 3B/3B+/3CM (64bit)
```

### Selected/unselected packages

```txt
Kernel modules --->
    Network Devices --->
        <*> kmod-r8169
    Other modules  --->
        <*> kmod-lunatik
    USB Support --->
        <*> kmod-usb-dwc2
        <*> kmod-usb-hid
        <*> kmod-usb2
        <*> kmod-usb3
Libraries --->
    <*> libssh
    <*> libssh2
    < > libustream-mbedtls
    <*> libustream-openssl    
LuCI --->
    1. Collections --->
        <*> luci
        <*> luci-ssl-openssl
Utilities --->
    <*> bcm27xx-userland
    <*> psmisc
    <*> usbutils
```
