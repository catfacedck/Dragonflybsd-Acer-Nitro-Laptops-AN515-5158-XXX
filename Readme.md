# Dragonflybsd 6.5 Snapshot Release on Acer Nitro AN515-51-XXX Series Laptops

These laptops typically come in a variety of hardware sku configurations:
  - AMD 7000 and above Ryzen cpus
  - Intel I12/I13/I14 -500/700/900 cpus
  - DDR4/DDR5 Ram
  - Thunderbolt 4
  - 165 Hz or 144 Hz display panels
  - AMD Radeon Graphics
  - Integrated Intel Graphics -P GT2 Iris Xe Graphics
  - Nvidia 30XX or 40xx Graphics
  - Integrated Intel WiFi 6/6E AX2XX series
  - Integrated Mediatek MT7921 WiFi or variants
  - Killer Ethernet (was Realtek now Intel)
  - Realtek and Nvidia Audio
  - Bluetooth >= 5.1
  - Integrated camera
  - Internel - two (2) nvme drive connectors and one (1) SATA drive connector - three (3) internel drives total.

**What you will get after installation:**
  - Ethernet network based dragonflybsd system using scfb Xorg driver, 1920x1080 165 Hz screen, mousepad, USB wireless mouse support, audio, mutiple drives, USB.

**What does not work:**
  - Intel integrated WiFi, Intel integrated Graphics (no drm-61-kmod -> no brightness/screen resolution control), Nvidia Graphics, suspend/sleep keys, camera.

>[!Note]
>This dragonfly installation was tested on Intel I5-12500 and Intel I7-13700 systems with Integrated Intel Graphics/Wifi, Nvidia 3050/4050 Graphics, and Killer Ethernet.
  

# **_Step-by step install procedure:_**

Typically these laptops are delived with Windows 11 installed on an internal 500 GB or 1 TB nvme drive. For this install it is far easier to accomplish on a new 2nd drive without the worry of screwing up the base Windows 11 installation. _Caveat lector._

However, in order to multiboot from the same or different drives one needs to use rEFind (https://www.rodsbooks.com/refind/).
Here the assumption is _at least_ two (2) drives: 1 with Windows 11 and 1 with dragonfly (and other) operating systems using rEFind.

>[!Caution]
>Before proceeding further, prepare a USB flash rescue/recovery disk for the existing drive 1 operating system, e.g. Windows 11, Ubuntu, Redhat, etc. Should rEFind 
 and/or boot troubles arise, this will save much time and allow system recovery. rEFind itself can be installed on a second USB flash for recovery purposes.

1) Prepare the laptop BIOS.
   ```
       a) Press F2 to enter the BIOS menu when powering the laptop on.
       b) In the MAIN menu:  F12 Boot Menu        Enabled
       c) In the BOOT menu:
                             Boot Mode            UEFI
                             Secure Boot          Disabled
       d) Press F10 to save and exit.
   ```
   
2) Change the laptop mousepad from iic to psm protocol. Mousepad iic protocol is not supported. To do so one must enter the BIOS advanced mode. Not planning to use the laptop mousepad? Skip this step.
   ```
    a) Press Fn+Tab three times. Reboot the laptop.
    b) Press F4, 4, R, V, F5, 5, T, G, B, F6, 6, Y, H, N while the laptop is turned off.
    c) Hold Fn+Tab while starting the computer, before entering the BIOS. Press F2 to enter the bios. Type ctrl-s.
    d) In the Main menu:
   			    Touchpad:		iic
    e) Change the iic value to psm:
   			    Touchpad:		psm
    f) Press F10 to save and exit.
   ```
   
3) Go get the dragonflybsd image to copy onto a USB flash from here: https://mirror-master.dragonflybsd.org/snapshots/x86_64/
   
   On Windows one can use Rufus (https://rufus.ie/en/) or dd from any Gnu/Linux or BSD system to flash the USB stick.
   A snapshot is necessary to get the Ethernet (re0) and audio drivers.
     ```
     Choose the DragonFly-x86_64-LATEST-BOOT.tar.bz2 file.
     Decompress this and flash to a USB stick. 
     ```

4) Test that the BIOS changes and USB stick are working.
   ```
     Plug in the USB stick and reboot the laptop.
     At power up when the keyboard lights flash, press F12.
     Select the USB stick from the menu to boot from it.
   ```
   
5) If the boot is successful inspect the dmesg boot log and pciconf output. If unsuccessful go look for the trouble.
   
   Do not install the system.

   ```
   Select: exit to shell and login as root.
   ```
   At the command prompt type:
   ```
   dmesg|less
   ```

   **Inspect the messages to ensure Ethernet was detected as well as the mousepad and audio. One must have Ethernet with the corresponding driver to continue 	      with the installation. The Ethernet hardware is attached to the pci bus. To view all the pci devices and their driver attachments.**
   
   At the command prompt type:
   ```
   pciconf -lvv|less
   ```
   
   No pci driver available is indicted by "none"
   Ethernet (re0 or other), mousepad (psm0), and audio (hdaa/hdacc) should have been detected.
   

7) Install drangonflybsd from the USB stick assumping the entire 2nd drive is used. Follow the directions here:
   
   Setup rEFind and drive partitions.
   ```
   https://www.dragonflybsd.org/docs/handbook/Installation/#index6h1
   ```
   
   Once the blank 2nd drive has partitions for rFEind and dragonflybsd, continue with the installation.
   Do a manual installation using the entire drive. Be sure to select the correct drive for the install!
   
   ```
   https://www.dragonflybsd.org/docs/handbook/Installation/#index3h1
   ```
   
8) After installation, reboot the laptop from the drive not the USB stick. When the keys flash select F12 for the rEFind boot menu. Select dragonflybsd (no    icon yet). Check that the /etc/fstab file looks something like this. The drive ID will be different matching dmesg.
   
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

   Check the default filesystem mountpoints. At the command prompt type:
   ```
   mount
   ```
   
   ```
   serno/210602802831.s5d on / (hammer2, local)
   devfs on /dev (devfs, nosymfollow, local)
   /dev/serno/210602802831.s5a on /boot (ufs, local)
   tmpfs on /tmp (tmpfs, local)
   tmpfs on /var/tmp (tmpfs, local)
   tmpfs on /var/run (tmpfs, local)
   tmpfs on /usr/obj (tmpfs, local)
   procfs on /proc (procfs, local)
   tmpfs on /var/run/shm (tmpfs, local)
   ```
   One should see something similar with a different drive ID after following the _manual installation using the entire drive_ instructions.
   Reboot the laptop. At the command prompt type:
   ```
   reboot
   ```
   When the keys flash select F12 for the rEFind boot menu. Select dragonflybsd (no icon yet).


10) Provision Ethernet networking. Be sure to connect the RJ-45 Ethernet port on the laptop to a work/home switch/router running DHCP using a suitable cable.

   At the command prompt type:
   ```
   ifconfig -a
   ```

   The pci Ethernet device should be shown (example here for re0). If not return to #5 above and look for the trouble.

   At the command prompt type:
   ```
   dhclient re0
   ifconfig -a
   ```

   This will assign an ip address from the router to the re0 interface and set up a static route.
   ```
     re0: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> metric 0 mtu 1500
	 options=1b<RXCSUM,TXCSUM,VLAN_MTU,VLAN_HWTAGGING>
	 ether 40:c2:ba:09:29:6b
     inet6 fe80::42c2:baff:fe09:296b%re0 prefixlen 64 scopeid 0x1
	 inet 192.168.0.101 netmask 0xffffff00 broadcast 192.168.0.255
	 media: Ethernet autoselect (100baseTX <full-duplex>)
	 status: active
   ```

   At the command prompt type:
   ```
   netstat -nr
   ```
   ```
   Routing tables

   Internet:
   Destination        Gateway            Flags    Refs      Use  Netif Expire
   Default            192.168.0.1        UGSc        2        4    re0       
   127.0.0.1          127.0.0.1          UH          0        0    lo0       
   192.168.0          link#1             UC          1        0    re0       
   192.168.0.1        64:20:e0:19:75:28  UHLW        3        0    re0   1198

   ```
   The re0 interface should be configured with a static route.

11) Provision package, dports, kernel source.

    At the command prompt type:

    ```
    cd /usr
    make pkg-bootstrap
    ```
    If up-to-date an error message will say "already exists". Ignore it. Test the pkg mechanism by installing some common shells which should access over the Internet     and install them.

    ```
    pkg install bash zsh
    ```

    Setup dports.
    ```
    make dports-create-shallow
    ```
    This creates dports in /usr for later use.

    Get the kernel source files.
    ```
    make src-create-shallow
    ```
    This creates the entire kernel and userland source tree /usr/src needed for development activities.

    Refer to the dragonflybsd web portal for instructions to build applications from dports and build custom kernels.
   
    https://www.dragonflybsd.org/docs/howtos/HowToDPorts/
   
    https://www.dragonflybsd.org/docs/handbook/ConfigureKernel/

12) Provision Xorg. Add the following content to /etc/sysctl.conf, /boot/loader.conf, and /etc/rc.conf .

    /etc/sysctl.conf
    ```
    # Needed for keyboard input in Xorg.
    kern.evdev.rcpt_mask=3
    # Switch the default sound to internal speakers.
    hw.snd.default_unit=1
    # Set the 0db sound reference level if volume is too low - tune by experiment.
    # hw.snd.vpc_0db=45

    ```
    /boot/loader.conf

    Line 3 drive ID will be different matching dmesg; this was added during installation.
    ```
    dm_load="YES"
    nvmm_load="YES"
    vfs.root.mountfrom="hammer2:serno/210602802831.s5d"
    ```

    /etc/rc.conf
    ```
    hostname="dflybsd65snap"
    clear_tmp_enable="YES"
    # Needed by Xorg.
    dbus_enable="YES"
    hald_enable="YES"
    tmpfs_var_run="YES"
    sendmail_enable="NONE"
    # Try some power flags. Must be tuned depending on laptop usage.
    powerd_enable="YES"
    powerd_flags="-u 7 -r 0.2 -T 60"
    # DHCP by default on interface re0.
    ifconfig_re0="dhcp"
 
    ```

    At the command prompt type:
    ```
    pkg install xorg
    pkg install xf86-input-evdev
    pkg install xf86-input-keyboard
    pkg install xf86-input-libinput
    pkg install xf86-input-mouse
    pkg install xf86-video-scfb
    pkg install chromium
    pkg install firefox
    ```

    The default fonts for Chromium and Firefox do not display correctly in particular for foreign languages.
    At the command prompt type:
    
    ```
    cd /usr/dports/x11-fonts/noto
    make install
    ```

    Test that Xorg, keyboard, mousepad and USB or another mouse are working. No Xorg configuration file is needed.
    At the command prompt type:
    ```
    startx
    ```

    An X session should start with three windows. Verify typing and mouspad/mouse movement in the windows.
    Exit the X session by typing "exit" or control-d in the Login window.
    If the screen freezes or no keyboard/mouspad control, try rebooting the laptop with the power button. 

    When the keys flash select F12 for the rEFind boot menu. Select dragonflybsd (no icon yet), boot and look for the trouble.
    

13) Provision audio and check the default audio device.

    At the command prompt type:

    ```
    cat /dev/sndstat
    ```
    
    ```
    Installed devices:
    pcm0: <NVIDIA (0x00a0) (HDMI/DP 8ch)> (play)
    pcm1: <Realtek (0x0287) (Internal Analog Speaker)> (play) default
    pcm2: <Realtek (0x0287) (Right Analog Headphones)> (play)

    ```

    This verifies that the audio devices were detected and have drivers attached. Internel speakers set as default as shown in /etc/sysctl.conf above.
    Test for sound. At the command prompt type:
    ```
    cat filename > /dev/dsp
    ```
    _filename_ cannot be empty. Static shoud be heard from the internal speakers. If not go look for the trouble (e.g. wrong sysctl default for internel 
    speakers, see #10 above).
    
    At the command prompt type:
    ```
    mixer
    ```
    ```
    Mixer vol      is currently set to  80:80
    Mixer pcm      is currently set to 100:100
    Mixer ogain    is currently set to 100:100
    ```
    This shows the audio volume and other parameters which can be set.
    
14) Provision a user and install xfce4 window manager. At the command prompt type:
    ```
    adduser
	Username: elephant
	Full name:             
	Uid (Leave empty for default): 
	Login group [elephant]: 
	Login group is elephant. Invite elephant into other groups? []: video wheel
	Login class [default]: 
	Shell (sh csh tcsh git-shell bash rbash zsh rzsh nologin) [sh]: zsh
	Home directory [/home/elephant]: 
	Home directory permissions (Leave empty for default): 
	Use password-based authentication? [yes]: 
	Use an empty password? (yes/no) [no]: 
	Use a random password? (yes/no) [no]: 
	Enter password: 
	Enter password again: 
	Lock out the account after creation? [no]: 
	Username   : elephant
	Password   : *****
	Full Name  : 
	Uid        : 1002
	Class      : 
	Groups     : elephant video wheel
	Home       : /home/elephant
	Home Mode  : 
	Shell      : /usr/local/bin/zsh
	Locked     : no
	OK? (yes/no): yes
    ```

    *_Ensure that user "elephant" has been added to the video and wheel groups in the /etc/group file (video for Xorg, wheel for root access)._*
    

    At the command prompt type:
    ```
    pkg install xfce
    pkg install xfce4-screenshooter-plugin
    ```
    The _screenshooter-plugin_ allows screenshots. Xfce4 and i3 window managers were tested on this hardware.

    Logout from the session. Login again as user "elephant". At the command prompt type:
    ```
    startxfce4
    ```
    The xfce4 window manager will start and can be configured to suit. xfce4 permits some provisioning of mouse actions. Consider using _moused_ in /etc/rc.conf for the mousepad.

    ![Screenshot_elephant](https://github.com/catfacedck/Dragonflybsd-on-Acer-Nitro-Laptop-AN515-51-XXX/assets/42676711/e7455ab5-352c-427b-8193-e0bb13a16dfc)


15) Provision _polkit_ so the *Shutdown* and *Log Out* menus may be used from xfce4. Add the following file to /usr/local/etc/polkit-1/rules directory.
```
polkit.addRule(function (action, subject) {
  if ((action.id == "org.freedesktop.consolekit.system.restart" ||
      action.id == "org.freedesktop.consolekit.system.stop")
      && subject.isInGroup("elephant")) {
    return polkit.Result.YES;
  }
});

polkit.addRule(function (action, subject) {
  if (action.id == "org.freedesktop.consolekit.system.suspend"
      && subject.isInGroup("elephant")) {
    return polkit.Result.YES;
  }
});
```
Name this file "80-shutdown-rules" (the number is not important but the filename must end with "rules"). The USER (elephant) here must be a member of the "operator" group in /etc/group.
>[!Caution]
>This is a critical security vulnerability.


14) Miscellaneous.
>[!Note]
> Todo: camera for work teleconferencing.
>
> Oracle Java for EDA tools (AMD/Xilinx, Intel/Altera, Synopsys/Cadence/Siemens, etc).
>
> WiFi and drm i915 Intel -P GT2 Iris Xe Graphics.


>[!Tip]
> October 2023/March 2024 *BSD/Linux tested on Acer Nitro 515-51-XXX series laptops running Intel.
>
> Openbsd 7.4: support for WiFi and Intel -P GT2 Iris Xe Graphics using drm. Nearly everything is functional (no camera) including iic mousepad, except suspend/resume. Likely a configuration issue. No Oracle Java.
>
> Freebsd 14-stable/15-current: support for WiFi and Intel -P GT2 Iris Xe Graphics using drm-61-kmod. Wifi is problematic sometimes hanging the system and/or dropping connections. Freebsd 14-release (or less) does not support Intel WiFi or Graphics on this platform. Suspend/resume is nonfunctional as is iic mousepad (use psm instead). No Oracle Java.
>
> Ubuntu 22.04/23.10: everything is functional (update usb file and recompile module for camera) including iic mousepad and EDA tools based on Oracle Java.

    

    

    
    
    
    
