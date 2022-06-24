# uno-220-poe
Advantech UNO-220-P4N2AE HAT community support

**Installing Ubuntu server**

1. Installing Ubuntu server: [how-to-install-ubuntu-on-your-raspberry-pi](https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi)
2. Enable Epson RX-8010SJ-B RTC, edit modules.conf to load rtc_rx8010 kernel module on boot:

`sudo nano /etc/modules-load.d/modules.conf`

> rtc_rx8010

3. Edit config.txt to enable TI TCA9554 IO expander:

`$ sudo nano /boot/firmware/config.txt`

> dtoverlay=pca953x,addr=0x27

4. Enable Infineon OPTIGA™ TPM SLB9670:

- Convert device tree blob back to device tree source:

`$ dtc -I dtb -O dts /boot/firmware/overlays/tpm-slb9670.dtbo -o tpm-slb9670-ce0.dts`

- Change "reg = <0x01>" to "reg = <0x00>":

`$ nano tpm-slb9670-ce0.dts`

- Convert device tree source to device tree blob:

`$ sudo dtc -I dts -O dtb tpm-slb9670-ce0.dts -o /boot/firmware/overlays/tpm-slb9670-ce0.dtbo`

- Add tpm-slb9670-ce0 overlay to config.txt:

`$ sudo nano /boot/firmware/config.txt`

> dtoverlay=tpm-slb9670-ce0

5. Set location timezone, i.e. Asia/Jakarta:

`$ sudo timedatectl set-timezone Asia/Jakarta`

6. Reboot

7. Get RTC time:

`$ sudo hwclock -r --verbose`

8. Set RTC time:

`$ sudo hwclock -w --verbose`

9. Test IO expander using gpiod tools, connect GPIO 0 to GPIO 1 terminal:

```
$ sudo apt-get install gpiod
$ gpiodetect
$ gpioinfo
$ gpioset 2 0=0
$ gpioget 2 1
$ gpioset 2 0=1
$ gpioget 2 1
```

10. To use IO expander in Node-RED using [node-red-contrib-tca9554](https://flows.nodered.org/node/node-red-contrib-tca9554), unload gpio_pca953x kernel module first or comment out "dtoverlay=pca953x,addr=0x27" in /boot/firmware/config.txt to disable kernel driver permanently:

`$ sudo modprobe -r gpio_pca953x`

12. Control PL1 GPIO LED using rpi gpio out node on PIN 32 (GPIO 12)

13. Test TPM support:

```
$ sudo apt-get install tpm2-tools
$ sudo tpm2_getrandom 8 | xxd -p
$ sudo tpm2_getrandom 16 | xxd -p
$ sudo tpm2_getrandom 32 | xxd -p
```

14. For serial console, connect RS-232/RS-485 USB serial converter and use minicom (Linux) or Tera Term (Windows) on host PC

- Run minicom on host PC, reboot Pi:
`$ minicom -D /dev/ttyUSB0 -b 115200 -con`

15. Delete "console=serial0,115200" in cmdline.txt to disable serial console and use RS-232/RS485 port for something else:

`$ sudo nano /boot/firmware/cmdline.txt`

- Install minicom on Pi and run it, also run minicom on host PC:
```
$ sudo apt-get install minicom
$ minicom -D /dev/ttyS0 -b 115200 -con
```
