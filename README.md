Credit goes to: jdwl1o1 for building this repository
Thanks to Bodhi, Luo and hippi-viking @ forum.doozan.com for dts & u-boot assistance.

# OpenWRT for Lenovo / Iomega ix2-dl
openwrt for the Lenovo / Iomega IX2-DL NAS

This project includes the files needed to compile OpenWRT for the Lenovo / Iomega ix2-dl NAS

details of openwrt are available at openwrt.org  

Files and images are provided 'as is' with no warranty or guarantees
This is a one way process. Once the NAND is overwritten, returning to stock will not be possible.

The information provided in this repository is for general informational purposes only. 
All information here is provided in good faith, however we make no representation or warranty of any kind, 
express or implied, regarding the accuracy, adequacy, validity, reliability, availability or completeness 
of any information.
Under no circumstance shall we have any liability to you for any loss or damage of any kind incurred 
as a result of the use of the code in this repository or reliance on any information provided. 
Your use of the code and your reliance on any information on this repository is solely at your own risk.

License is GPL2, unless otherwise specified in the files. No ownership is claimed over work of others.


Lenovo ix2-dl is a dual SATA NAS powered by a Marvell
 Kirkwood SoC clocked at 1.6GHz. It has 256MB of RAM and 1GiB of
 flash memory (NAND), 1x USB 2.0 and 1x 1Gbit/s NIC. 
 Was also sold by Iomega as the Iomega StorCentre ix2-dl.
 This device is very similar to the ix2-200 however it lacks 2 USB ports and includes 
 a larger nand flash unit. The IX2-DL was sold without drives, with the OS stored on 1GB NAND.
```
Specification:
- SoC: Marvell Kirkwood 88F6282
- CPU/Speed: 1600Mhz
- Flash-Chip: Samsung NAND
- Flash size: nand: 1024 MiB, SLC, erase size: 128 KiB, page size: 2048, OOB size: 64
- RAM: 256MB
- LAN: 1x 1000 Mbps Ethernet
- WiFi: none
- 1x USB 2.0
- UART1: for serial console. UART0 not available.
```
Installation instructions (modified from the ix2-200): 
This runs OpenWrt from ram to allow sysupgrade to install to nand.

1. Download initramfs-uImage and sysupgrade images, copy them on your tftp server (or ext2 formatted usb drive)
(i.e `openwrt-21-02-defconfig-kirkwood-lenovo_ix2-dl-initramfs-uImage` and `openwrt-21-02-full-kirkwood-lenovo_ix2-dl-squashfs-sysupgrade.bin`)

2. After power on, while connected over console (115200, N, 8, 1), hit any key to stop the boot process. 
This is the uboot environment, run the following commands, one at the time:
```
    setenv mainlineLinux yes
    setenv console 'console=ttyS0,115200'
    setenv mtdparts 'mtdparts=orion_nand:0x100000@0x000000(u-boot)ro,0x20000@0xA0000(u-boot environment)ro,0x300000@0x100000(kernel),0x1C00000@0x400000(ubi)'
    setenv owrtboot 'nand read.e 0x800000 0x100000 0x300000;; setenv bootargs $(console) $(owrt_bootargs_root); bootm 0x800000'
    setenv owrt_bootargs_root 'root='
    setenv bootcmd 'run owrtboot'
    saveenv
```
For USB Boot:   
```
 usb reset; ext2load usb 0:1 0x00800000 /[initramfs image]; bootm 0x00800000
```
If you boot from a TFTP server (recommended):
```
    setenv serverip [tftp server ip]    
    setenv ipaddr 192.168.1.13
    tftpboot 0x00800000 [initramfs image]
    bootm 0x00800000
```
replace [tftp server ip]  with your TFTP server
replace [initramfs image] with the exact name of the file that is on the TFTP server (i.e `openwrt-21-02-defconfig-kirkwood-lenovo_ix2-dl-initramfs-uImage`)
After it loads the initramfs image, you need to transfer the `sysupgrade` image. You can use `scp` from the TFTP server to the IX2-DL, like so:
```
scp openwrt-21-02-full-kirkwood-lenovo_ix2-dl-squashfs-sysupgrade.bin root@opewwrt.local:/mnt
```
3. You can continue on the console, or ssh to `openwrt.local` and sysupgrade to install into flash.
If using USB flash:
```
    mkdir /mnt/usb
    mount /dev/sda1 /mnt/usb
```
if using TFTP method
```
    cp /mnt/openwrt-21-02-full-kirkwood-lenovo_ix2-dl-squashfs-sysupgrade.bin /tmp   
    sysupgrade -n /tmp/openwrt-21-02-full-kirkwood-lenovo_ix2-dl-squashfs-sysupgrade.bin
```
4. access openwrt.local by dhcp ip address assigned by your router or at 192.168.1.1 if connected directly with ethernet to your PC/mac:

Notes:
 - SATA drives should not be inserted when installing the OperWRT operating system
 - any data on the drives will not be accessible until `LVM` and `mdadm` modules are installed.
 - this is a 1 way process. Once the nand is overwritten returning to stock will not be possible.
 - to access the u-boot variables from within openwrt, add the following lines to /etc/fw_env.config in openwrt
```
/dev/mtd1 0x0000 0x20000 0x20000
/dev/mtd2 0x0000 0x20000 0x20000
```
- after accessing openwrt.local webinterface, navigate to `System` - `Software`, click on `Configure OPKG...`
  then in the section `/etc/opkg/distfeeds.conf` replace with following:
```
  src/gz openwrt_core https://mirror-03.infra.openwrt.org/releases/21.02-SNAPSHOT/targets/kirkwood/generic/packages/
  src/gz openwrt_base https://mirror-03.infra.openwrt.org/releases/21.02-SNAPSHOT/packages/arm_xscale/base
  src/gz openwrt_luci https://mirror-03.infra.openwrt.org/releases/21.02-SNAPSHOT/packages/arm_xscale/luci
  src/gz openwrt_packages https://mirror-03.infra.openwrt.org/releases/21.02-SNAPSHOT/packages/arm_xscale/packages
  src/gz openwrt_routing https://mirror-03.infra.openwrt.org/releases/21.02-SNAPSHOT/packages/arm_xscale/routing
  src/gz openwrt_telephony https://mirror-03.infra.openwrt.org/releases/21.02-SNAPSHOT/packages/arm_xscale/telephony
```
- click `Save`, then `Update Lists...`

