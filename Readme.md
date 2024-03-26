# Dragonflybsd 6.5 Snapshot Release on Acer Nitro AN515-51-XXX Series Laptops

These laptops typically come in a variety of hardware sku configurations:
  - AMD 7000 and above Ryzen cpus
  - Intel I12/I13/I14 -500/700/900 cpus
  - 165 Hz or 144 Hz display panels
  - AMD Radeon Graphics
  - Integrated Intel Graphics -P GT2 Iris Xe Graphics
  - Nvidia 30XX or 40xx Graphics
  - Integrated Intel WiFi 6 AX2XX series
  - Integrated Mediatek MT7921 WiFi or variants
  - Killer Ethernet (was Realtek now Intel)
  - Realtek and Nvidia Audio
  - Internel - two (2) nvme drive connectors and one (1) SATA drive connector - three (3) internel drives total.

What you will get after installation:
  - Ethernet network based dragonflybsd system using scfb Xorg driver, 1920x1080 165 Hz screen, mousepad, USB wireless mouse support, audio, mutiple drives, USB.

What does not work:
  - Intel integrated WiFi, Intel integrated Graphics (no drm-61-kmod -> no brightness/screen resolution control), Nvidia Graphics, suspend/sleep keys.

Note: This dragonfly installation was tested on Intel I5-12500 and Intel I7-13700 systems with Integrated Intel Graphics/Wifi, Nvidia 3050/4050 Graphics, and Killer Ethernet.
  

# **_Step-by step intall procedure:_**

Typically these laptops are delived with Windows 11 installed on an internal 500 GB or 1 TB nvme drive. For this install it is far easier to accomplish on a new 2nd drive without the worry of screwing up the base Windows 11 installation. _Caveat lector._

However, in order to multiboot from the same or different drives one needs to use rEFind (https://www.rodsbooks.com/refind/).
Here the assumption is two (2) drives: 1 with Windows 11 and 1 with dragonfly (and other) operating systems using rEFind.

1) Prepare the laptop BIOS.
   ```
       a) Press F2 to enter the BIOS menu when powering the laptop on.
       b) In the MAIN menu:  F12 Boot Menu        Enabled
       c) In the BOOT menu:
                             Boot Mode            UEFI
                             Secure Boot          Disabled
       d) Press F10 to exit.
   ```
   
2) Change the laptop mousepad from iic to psm protocol. Mousepad iic protocal is not supported. To do so one must enter the BIOS advanced mode. Not planning to use the laptop mousepad? Skip this step.
   ```
   ```
   
3) Go get the dragonflybsd image to copy onto a USB stick from here: https://mirror-master.dragonflybsd.org/snapshots/x86_64/
   
   On Windows one can use Rufus (https://rufus.ie/en/) or dd from any Gnu/Linux or BSD system to flash the USB stick.
   A snapshot is necessary to get the Ethernet (re0) and audio drivers.
     ```
     Choose the DragonFly-x86_64-LATEST-BOOT.tar.bz2 file.
     Decompress this and flash to a USB stick. 
     ```

5) Test that the BIOS changes and USB stick are working.
   ```
     Plug in the USB stick and reboot the laptop.
     At power up when the keyboard lights flash, press F12.
     Select the USB stick from the menu to boot from it.
   ```
6) If the boot is successful inspect the dmesg boot log. If unsuccessful go look for the trouble.
   ```
   Do not install the system.
   
   Select: exit to shell and login as root.
   At the command prompt type: dmesg|less
   
   Inspect the messages to ensure Ethernet was detected as well as the mousepad.
   One must have Ethernet and driver to continue with the installation.
   
   View all the pci devices and their driver attachments by typing: pciconf -lvv|less
   No pci driver available is indicted by "none"

   Ethernet (re0 or other), mousepad (psm0), and audio (hdaa/hdacc) should have been detected.
   ```

7) Install drangonflybsd from the USB stick assumping the entire 2nd drive is used. Follow the directions here:
   ```
   Setup rEFind and drive partitions.
   ```
   https://www.dragonflybsd.org/docs/handbook/Installation/#index6h1
   ```
   Once the blank 2nd drive has partitions for rFEind and dragonflybsd, continue with the installation
   Do a manual installation using the entire disk.
   ```
   https://www.dragonflybsd.org/docs/handbook/Installation/#index3h1

   After installation, check that the /etc/fstab file looks something like this. (drive ID will be different matching dmesg).
   ```
    serno/210602802831.s5a		/boot	ufs	rw	1	1
    serno/210602802831.s5b		none	swap	sw	0	0
    serno/210602802831.s5d		/	hammer2	rw	1	1
    # Add these 5 lines yourself.
    dummy /tmp tmpfs rw 0 0
    dummy /var/tmp tmpfs rw 0 0
    dummy /var/run tmpfs rw,-C 0 0
    dummy /usr/obj tmpfs rw 0 0
    proc /proc procfs rw 0 0
   ```
   
8) Provision Ethernet networking. Be sure to connect the RJ-45 Ethernet port on the laptop to a work/home router running DHCP using a suitable cable.
   ```
   Boot the laptop and login as root.
   At the command prompt type: ifconfig -a

   The pci Ethernet device should be shown (example here for re0). If not return to #6 above and look for the trouble.

   At the command prompt type:
   dhclient re0

   This will assign an ip address from the router to the re0 interface and set up a static route.
   ```

9) Update package, dports, kernel source.
    ```

    ```

10) Install common shells to test pkg:
    ```
    pkg install bash zsh
    ```

11) Provision Xorg. Add the following content to /etc/sysctl.conf, /boot/loader.conf, and /etc/rc.conf .
    ```
    # Needed for keyboard input in Xorg.
    kern.evdev.rcpt_mask=3
    # Switch the default sound to internal speakers.
    hw.snd.default_unit=1
    # Set the 0db sound reference level if volume is too low - tune by experiment.
    # hw.snd.vpc_0db=45

    ```
    /boot/loader.conf (drive ID will be different matching dmesg)
    ```
    dm_load="YES"
    nvmm_load="YES"
    ```

  /etc/rc.conf
  ```
  hostname="dflybsd65snap"
  clear_tmp_enable="YES"
  dbus_enable="YES"
  hald_enable="YES"
  tmpfs_var_run="YES"
  sendmail_enable="NONE"
  powerd_enable="YES"
  powerd_flags="-a hiadaptive -b adaptive"
  ifconfig_re0="dhcp"
 
  ```

13) Provision audio. The default audio device and 0db volume.
    ```
    At the command prompt type: cat /dev/sndstat
    ```
