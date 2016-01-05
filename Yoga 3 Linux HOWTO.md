Installing Ubuntu 14.10 on Yoga 3 Pro
=====================================

With this intructions you get a Ubuntu 14.10 (KoljaWindeler: Or LinuxMint 17.10) on a Yoga 3 Pro from Leonvo with full Secure Boot enabled UEFI mode. This means dual boot to Windows 8 (KoljaWindeler: And Windows 10) works just fine. When you turn on the Yoga Grub will greet you and you can choose to boot Ubuntu (the default) or Windows. All relevant components work good enough to use this platform as your main on the go workstation including Wifi, Bluetooth, touch pad and touch screen.

# Create recovery USB flash drive

If you ever want to restore the original partition layout and the recovery partition make sure to not forget this step. If you have a recovery USB flash drive already (for Yoga 3) you can skip this step.

Requirements: USB flash drive with at least 16GB capacity. All data will be overwritten.

1. Boot into Windows 8.1

2. Do not connect to Internet and create a local account.

3. Open Control Panel

4. Select Recovery

5. Select Advanced recovery tools

6. Select Create a recovery drive

7. Select "Copy the recovery partition from the PC to the recovery drive" checkbox

8. Select the USB drive from the list

9. Continue and wait for the process to finish. It takes about half an hour.

With this USB flash drive you can later reset to the default content of the SSD including the recovery partition. (Boot from USB, Troubleshoot, Reset your PC, Next, Yes to re-partition, Fully clean the drive, Reset).

# Shrink Windows partition (Dual boot)

The partition can be shrinked with the Windows partition manager. It is important that you do this as early as possible to avoid long defragmentation. Shrink the Windows partition so it has around 100GB capacity (eg. shrink by 340000MB for a 512GB drive).

1. Shrink Windows partition to around 100GB

2. Make sure there is empty space (330 GB) after shrink.

Shrinking should be quick / take no time.

# Disable Windows8 FastStartup (Dual boot)

To boot another OS with EFI FastStartup of Windows8 must be disabled.

1. Open Control Panel

2. Go to Power Options -> System Settings

3. Enable Administrator access (click on "Change settings that are currently unavailable")

4. Uncheck (Turn on fast startup)

5. Save changes

# BIOS Settings

To enter the BIOS shut down the Yoga 3. Then long press the small button (with a pen or something). The button is called the Novo button and is located next to the power button on the right. When long pressing this button, the Yoga 3 will turn on and show the boot menu, which includes an option to enter the BIOS.

Check the following BIOS settings:

1. Secure Boot [Enabled]

2. Boot Mode [UEFI]

3. Fast Boot [Enabled]

4. USB Boot [Enabled]

# Boot from USB flash drive (Ubuntu 14.10)

Prepare a flash dive with Ubuntu 14.10 or later image. Use the Novo button to get into boot menu and select the "EFI USB Device". This should show Grub - select “Try Ubuntu without installing”.

1. Boot Ubuntu 14.10 USB flash drive

2. Try Ubuntu without installing

3. Test Ubuntu to see if everything works

    - Check touchpad (works)

    - Check touchscreen (works)

    - Check keyboard (works)

    - Check screen dim (works)

    - Check audio (works)

    - Check Wifi (not working)

# Wifi

Wifi is not working out of the box as it needs a proprietary driver. The chip is a BCM4352 from [Broadcom](http://www.broadcom.com/support/802.11/linux_sta.php). To get Wifi working just install 'bcmwl-kernel-source' Ubuntu package is available on the installation image. This package bundles the Broadcom proprietary binary driver as DKMS module.

	sudo apt-get install bcmwl-kernel-source

 (KoljaWindeler: e.g. Connect your Android phone, enable WiFi and USB tethering and plug it into the yoga. You're online within no-time.) After that the module should be compiled and loaded. Check with dmesg (wlan0 should appear). 
 
On Ubuntu 14.10 but not on Linux Mint 17.10:
 We now have a wifi0 interface but it is blocked by hardware switch. This is because the ideapad-laptop module is broken in the default kernel and needs a patch ([http://ubuntuforums.org/showthread.php?t=2215044](http://ubuntuforums.org/showthread.php?t=2215044)) as the Yoga’s do not have Wifi kill switches. Eventually there will be a Ubuntu installer image with a fixed Kernel, but for now just remove the module.

	sudo modprobe -r ideapad_laptop

Voila, Wifi suddenly works like a charm and you can connect using network manager. As mentioned before, this problem has been fixed in later Kernel revisions - so removing the module is only required when installing.

# Install Ubuntu 14.10

Make sure you have booted with EFI by checking if '/sys/firmware/efi' exists. Do not continue if it does not exist and reboot in EFI mode (Dual boot). Ubuntu can be installed on the SSD now. Click on Install Ubuntu and make the following selections:

1. Something else (As we want full disk encryption)

2. In the free space, create partition 500MB ext4 /boot

3. In the remaining space, create a partition "Use as physical volume for encryption"

4. The newly created partition is created under /dev/mapper. Select this partition and click "Change…" button and choose ext4 /

5. As device for boot loader installation select the EFI partition (probably /dev/sda2)

6. We do not create a swap partition. So click on "Install Now"

7. Select Timezone and Keyboard

8. Create your user. Do not select to encrypt the home folder if you are the only user on this machine

Wait until the system is installed. This should take around 5 minutes. When done, restart the system. It did not reboot for me .. so hard reset.

The system should now boot into Grub. There is a option to boot into Windows there too - so test this first (works). Then restart and boot into Ubuntu.

Remember that Wifi will not work, until the drivers are installed. Luckily the driver is on the installation image (plug it in again), we just need to install it into the installation image with dpkg.

1. Mount USB installation image from Files

2. Install dkms from 'pool/main/d/dkms'

3. Install bcmwl-kernel-source from 'pool/restricted/b/bcmwl'

4. Blacklisting is not required anymore, as the problem was fixed in Kernel 3.16.0-29 - check with "rfkill list all".

Voila - we are complete.

# Bluetooth Firmware

 (KoljaWindeler: The Bluetooth will work out-of-the-box on LinuxMint 17.10)
 
The firmware for the Bluetooth controller is not available in the Linux firmware package. We can create the firmware from the existing Windows driver. So mount the Windows partition and search for BCM20702A1_001.002.014.1443.1572.hex. This is the file we have to convert to a binary HCD file to be loaded by the Linux Kernel. Fortunately there is the tool [hex2hcd](https://github.com/jessesung/hex2hcd) for this.

	git clone https://github.com/jessesung/hex2hcd.git
	cd hex2hcd
	make
	./hex2hcd BCM20702A1_001.002.014.1443.1572.hex BCM20702A0-0489-e07a.hcd
	sudo cp BCM20702A0-0489-e07a.hcd /lib/firmware/brcm
	reboot

Now you can enable Bluetooth from the tray icon. To check in the terminal, run `hciconfig dev`. Your device will appear there as UP.

# HiDPI support

Ubuntu supports scaling up the Unity interface in System Settings -> Displays. I use 1.5 as scale factor. In Firefox, go to about:config and set 'layout.css.devPixelsPerPx' to 2.
 (KoljaWindeler: Linux Mint 17.10 -> system settings -> general -> select HDPI and REBOOT!)

# Dual screen / external display

The external display works out of the box, including 4K support through the micro HDMI port. Ubuntu 14.10 comes with Kernel 3.16 which has the latest Intel drivers. So at time of writing there is no better Intel driver. See known issues.

# Improve battery life

TLP is an advanced power management tool for Linux that automatically handles settings and tweaks to enhance the battery live and power saving. It does not have a GUI and just runs in the background.

	sudo apt-add-repository ppa:linrunner/tlp
	sudo apt-get update
	sudo apt-get install tlp tlp-rdw smartmontools ethtool powertop
	sudo tlp start

## Generic Linux kernel tweaks: Modifying Grub boot config

Linux kernel has a power regression problem.  See http://www.phoronix.com/scan.php?page=article&item=intel_i915_power 

This fix tweaks the Grub bootloader and helps extend battery life.  Edit your grub config as follow:

    $ sudo vim /etc/default/grub

Replace `GRUB_CMDLINE_LINUX_DEFAULT`:

    # GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
    GRUB_CMDLINE_LINUX_DEFAULT="quiet splash pcie_aspm i915.lvds_downclock=1 i915.i915_enable_rc6=1 i915.i915_enable_fbc=1"

Sync Grub

    $ sudo update-grub
    $ sudo reboot


# Known issues

* Sometimes hangs on shutdown / reboot. This happens with Windows too and seems to be fixed by upgrading the Yoga 3 to the latest firmware.

* Intel platform drivers have various issues. So the rule is to get the newest ones available. This involves installing the [latest Intel GPU drivers](https://01.org/linuxgraphics/downloads) which are not directly available for Ubuntu 14.10 yet and manually installing the xserver-xorg-video-intel package from Ubuntu 15.04 (Vivid). This helps to improve the overall experience.

* HiDPI with external displays is a little flaky and can result in strange redraw issues. Also the Intel platform driver.

* Camera quality is terrible and blurry. The camera is just this bad - it actually a shame to put such a crappy camera into a device with pro in its name. It also does only do 720p max.

* Fan does run all the time at low speed. No way to read fan speed nor control it.

* Kernel 3.16.0-30 has some issues with the Broadwell GPU platform. Kernel 3.18 or 3.19 should improve the situation. See [here](http://www.phoronix.com/scan.php?page=news_item&px=MTc1ODg) and [here](http://www.phoronix.com/scan.php?page=news_item&px=MTc1ODM) for details. I started building my own Kernel - see the instructions at https://github.com/longsleep/linux_patches/tree/yoga3pro

* Broadcom Wifi driver is not open source and needs patches to compile with latest Kernels (3.17 or later).

* Bluetooth firmare is not available directly and needs to be converted from Windows.

* Thermald automatic configuration is buggy and returns all kind of unsupported zones. Thus thermald does not do anything useful. Do not use it for now.

* Static noise in headphone jack when rebooting from Windows. Can be avoided by doing a complete shutdown instead of reboot.

# Extra steps after installing Ubuntu

Some things you should do after installing Ubuntu to get the most out of your installation.

* Turn off the online search. Open System Settings and head to Security & Privacy. In the Search tab turn off the online search results.

* Install restricted extras 'sudo apt-get install ubuntu-restricted-extras'.

* Remove Apport 'sudo apt-get remove apport' to avoid leaking private information / turns of bug reports.

* Install tweak tools 'sudo apt-get install unity-tweak-tool'. Use the Unity tweak tool to enable multiple desktops (i use 3x3).

Have fun with the Yoga 3 Pro on Linux.

# Use Kernel 4.0

I recommend to run the Yoga 3 Pro with the latest available Linux Kernel as 4.0 brings lots of hanges and improvements for the Intel platform. I have a Kernel pachtes tree and configuration for Yoga 3 Pro in [my Kernel patches](https://github.com/longsleep/linux_patches/tree/yoga3pro) repository. Please note that this requires to compile the Wifi driver as well, so it is compatible with Kernel 4.0.

# My result / recommendation

For me the Yoga 3 Pro did not not work. I stopped using it. It was mostly the keyboard, the constant fan noise and CPU speed.

### Pro's

  + Awesome SSD with 512GB at lightning speed.
  + Nice HiDPI screen.
  + 2 USB3 connectors.
  + Long battery life.
  + Secure boot.
  + Very thin and light.

### Con's

  - Constant fan noise.
  - Thermal design is a fail. CPU gets hot quickly and thus is slow. Too slow.
  - Keyboard layout and feeling is very bad for me. I cannot type on this thing.
  - Very bad camera.
  - Intel platform drivers have all kinds of issues. See [known issues](https://01.org/linuxgraphics/downloads/2014/2014q4-intel-graphics-stack-release). Lets hope it improves with the first 2015 release.
  - No drivers for advances power and battery settings on Linux.
  - Wifi driver is not open source / in standard kernel.
  - Quite large / wide.
  - Touchpad clicks too easy.
  - Large USB charging plug on the left.
  - Hard to open, essentially two hands required.


# More things to investigate

* Own the platform by adding own platform key for UEFI secure boot

* Intel DPTF drivers (check out for Chromium OS)

* Explore possible Kernel patches to be able to turn of the fan.

* Get more out of the battery by adding support for the modes supported on Windows.

--
Simon Eisenmann
