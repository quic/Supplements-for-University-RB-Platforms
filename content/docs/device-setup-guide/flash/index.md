+++
title = 'Flash the Image'
date = 2024-09-24T14:32:02-07:00
draft = false
weight = 3
+++
## Set up udev rules
udev manages access to the RB5 device when it is plugged in. We need to add a rule to udev to give normal users access to the device.

1. Download the [udev rules file](51-qcom-usb.rules)
2. Move the file to the udev rules directory
```sh
sudo cp 51-qcom-usb.rules /etc/udev/rules.d
```
3. Reload udev rules
```sh
sudo udevadm control --reload-rules
```
```sh
sudo udevadm trigger
```

## Put RB5 into QDL mode
QDL mode allows a new image to be written to the RB5. It is sometimes also called *9008 mode*

![Force USB Boot Button](force-usb-boot.png "Force USB Boot Button")
![12V Power Port](12v-power-port.png "12V Power Port")

1. Hold the *Force USB Boot* button on the device
2. Insert the 12V power cable. You can release the button after the power cable is inserted.
3. Connect the RB5 device to the host system with the USB-C cable.
4. Verify that the device shows up by running the following on the host
```sh
lsusb | grep 9008
# example output:
# Bus 002 Device 002: ID 05c6:9008 Qualcomm, Inc. Gobi Wireless Modem (QDL mode)
```

## Flash with QDL
QDL is the tool for flashing images to the device. From Ubuntu 24.04 onwards, qdl is available in the official apt repos. For earlier versions of Ubuntu, the package in the snap repos can be used. Alternatively, there is a version using docker that can work on other distributions.

1. Unzip the image downloaded in the previous steps. `QRB5165.LU2.0_20240603.2127_debug` will be used as an example.
```sh
unzip QRB5165.LU2.0_20240603.2127_debug.zip
```
```sh
cd QRB5165.LU2.0_20240603.2127_debug/full_build/ufs
```
2. Install the qdl tool on the host and flash.
{{< tabs items="Apt,Snap,Docker" >}}
  {{< tab >}}
  ```sh
  sudo apt install qdl
  ```
  ```sh
  qdl ./prog_firehose_ddr.elf rawprogram*.xml patch*.xml
  ```
  {{< /tab >}}
  {{< tab >}}
  ```sh
  sudo snap install qdl
  ```
  ```sh
  qdl ./prog_firehose_ddr.elf rawprogram*.xml patch*.xml
  ```
  {{< /tab >}}
  {{< tab >}}
  3. Download the [Dockerfile](Dockerfile) and build the image
  ```sh
  docker build -t qdl .
  ```
  4. Flash
  ```sh
  docker run --privileged -v /dev/bus/usb:/dev/bus/usb -v .:/build qdl:latest
  ```
  {{< /tab >}}
{{< /tabs >}}

## Power on the device

![Power Button](power-button.png "Power Button")

To power on the RB5 device, hold the power button for one second, then let go. You should see a green LED on the top of the device turn on.
