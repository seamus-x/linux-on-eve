# linux-on-eve
Linux configuration files and utility scripts for Google Pixelbook (EVE)

## Audience
Pixelbook recently got an upgrade to Linux kernel 5.4 with Chrome OS R89, and I thought I'd mark the occasion by sharing my take on the whole Pixelbook Linux experiment. With a dozens of releases since the launch of Pixelbook behind us, some of the instructions and scripts previously posted online no longer work, although this git is heavily informed and built on these prior works (see references). The goal of this git is to 

There are many benefits to running Linux natively on Pixelbook. Not only does it offer a fully fleged desktop experience for developers and consumers alike (I can haz software?), compared to Linux (Beta) aka Crostini, it is much faster, and compared to Crouton, which must be run in a chroot in Developer Mode, native Linux is more secure and more customizable without limitations of chroot (even though I must admit that [Crouton + Sommelier](https://github.com/jcdang/chromeos-ubuntu-sommelier) is a pretty dope setup). This git offers some useful configuration files and utility scripts with a supplemental tutorial (this page) that can help make running Linux on Pixelbook i-7 a seamless experience. 

Readers are assumed to already know how to turn on Developer Mode in Chrome OS, flash Pixelbook BIOS, and install Linux natively on Pixelbook. If not, please visit https://mrchromebox.tech for more information, and [this git](https://github.com/yusefnapora/pixelbook-linux) has an excellent tutorial (which also informs this git) on how to get started.

## Features
The setup provided herein fixes the following potential issues with a Debian-based distro running on Pixelbook out-of-box with stock kernel and applications:
Item | Issue | Fix | Default Outcome
-----| ----- | ----- | -----
Screen Backlight | Not dimmable | Custom kernel | Adjustable backlight through desktop builtin control
Touchpad | Nonfunctional or awkward control | Custom kernel | Fully functional
Touchscreen | Nonfunctional | Custom kernel | Fully functional
Wifi | Failure to detect device at boot or wakeup | Custom kernel | Fully functional
Sound | No sound, poor control, peripherals not recognized | ALSA plugin (cras or hw) | Fully functional capture and playback with basic controls, automation script for peripheral control
USB | Not recognizing devices, shoddy connection | Firmware update | Fully functioning USB ports as experienced with ChromeOS
Keyboard | Lacking several keys | Custom key mapping | Redundant keys remapped (customizable)

The kernel configuration in the default setup additionally meets the the kernel requirements for a host of common applications and functions. While the default custom kernel may not be the leanest, it is relatively free of bloat. TTY emulation and USB serial are enabled for rescue and on-device debugging. Both `zram` and disk-based swap are enabled (no partition swap, unfortunately). The default setup is also capable of running `libvirt-qemu` and `lxd` (Windows 10 runs very well as a `kvm` on Pixelbook).
> Note: network filtering and filesystem support is kept at a minimum, additional kernel options may need to be enabled for certain virtualization and network/firewall configurations.

## Outstanding issues
* **USB fix is experimental with watchdog disabled in the custom firmware; this may potentially cause issues down the line, and has a noticeable power consumption impact likely due to a high amount of IRQs.**
* Laptop display is inverted in some desktop environment, multiple display setup, and tablet mode, likely due to miscommunication with sensors (accelerometer, gyroscope, and/or lid angle). There are a couple easy fixes for errant screen rotation in desktop mode if it presents, and it's easy to set up an `xrandr` automation script to rotate screen if you're using X11, but tablet mode rotation issue in Wayland has still yet to be resolved (not a big deal if you're sticking with X11 or if you're willing to give up the 2-in-1 feature of Pixelbook).
* Google Pen is recognized as a mouse instead of a stylus.
* Some kernel error messages with no known or significant functional impact. A couple at boot, and a few recurring ones related to sound.

## Compatibility

Files contained herein are devoloped and tested in the following environment:
- Hardware: Pixelbook i-7 (eve)
* Kernel:   Linux 5.4.104 for Chromium OS (R90)
* OS:       Debian 11 (`bullseye`) or Ubuntu 20.04LTS (`focal`)
* Desktop:  GNOME 3.36 or 3.38

It is my hope that this git can inform Chromebook Linux enthusiasts in any way big or small regardless of their hardware and distribution choices, and while these are not hard requirements, they are the recommended setup to fully take advantage of this git. Compatibility of this git on your device and distro will vary, as discussed below:

### Hardware

A lot of issues and subsequent fixes with Linux on Pixelbook has to do with the peculiarities of EVE hardware, which lacks official Linux support. If you're using a different Chromebook, chances are that it is already better supported by common Linux distros. If you're using a Pixelbook, i-3 and i-5 flavors may not have enough computing power to run GNOME smoothly, which can be a resource hog, though this git is desktop-environment-agnostic. If you're using a Pixelbook Go, the configurations here won't work properly without modifications, though you may still benefit from revewing them for a jumpstart.

### Kernel

Linux kernel v5.4 for Chromium OS is strongly recommended, as it is the basis for the default custom kernel in this git. Certain things just won't work without using the Chromium-flavored Linux kernel, most notably screen backlight. Kernel v5.4 has been used by Chrome OS on Pixelbook since R89 (v4.4 prior to that). Kernel v4.4 works well with Debian 10, but lacks GPU acceleration in Debian 11 (likely due to a `mesa` bug).

### OS

Debian 11 or Ubuntu 20.04LTS is recommended. Instructions here should work with few to no modification on Debian-based operating systems that use `systemd`, which also includes Gallium OS, among others. For non-Debian-based distros, parts of the instructions should work, other portions may need significant modification.

Debian 11 `bullseye` is the tested OS for this git and it works great out of box on Pixelbook i-7. Compared to Debian 10 `buster`, Ubuntu (both 18.04LTS and 20.04LTS), and even GalliumOS, `bullseye` has excellent support for eve touchpad, keyboard backlight, and HiDpi display (thanks to an upgrade to GNOME 3.38). However, like with the other distros and as noted above, without the Chromium-flavored kernel and proper configuration, several key interfaces do not work properly, most notably sound, screen backlight, and USB ports. 

## Additional Requirements
It is strongly recommended that you acquire a [SuzyQ USB debugging cable (SuzyQable)](link) before proceeding. Not only does it make the flashing process a lot easier (and recoverable), if you want functional USB ports, you'll have to have one in order to flash certain component firmware.

## Tasks
### Checklist
- [x] Read Sections Compatibility and Additional Requirements
- [ ] Optional preparation
- [ ] Install MrChromebox's firmware
- [ ] Install Linux
- [ ] Build custom kernel
- [ ] Customize keyboard
- [ ] Fix sound
- [ ] Fix USB
- [ ] Fix screen rotation

### Optional Preparation

It is recommend that you fully upgrade your Chrome OS and make an image copy of the Chrome OS root filesystem before flashing BIOS and installing Linux, as it contains the most up-to-date configurations files. Absent that step, a Chrome Recovery Image would work as well, although it normally lags a couple releases behind. If you're coming up short of both and already running Linux on your Pixelbook, there is no need to revert to Chrome OS; you can always make a Chrome Recovery Media using the Google Chrome browser's Chrome Recovery Utility tool, or download it directly from Google.
To make a copy of the root filesystem, enable Developer Mode, bring up `crosh` (<kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>T</kbd>) and enter a Linux terminal emulator by typing `shell` <kbd>Enter</kbd>. Alternatively, access virtual terminal (<kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>F3</kbd>) and log in using `XXXXXXXXXXXXX` (no password required, if you have not previously set one - you see how unsecure using Crouton is now?). Make sure you have at least 4GB of free space on your harddrive (or use a USB drive). Chrome OS uses A/B root system for backup and recovery purposes, the more recent release can exist on either `/dev/nvme0n1p3` or `/dev/nvme0n1p5`, and we want the most recent one (R90-13816 at the time of this writing). To check, mount each partition with
```
mkdir /tmp/root-a
mkdir /tmp/root-b
sudo mount -t ext2 -o ro /dev/nvme0n1p3 /tmp/root-a
sudo mount -t ext2 -o ro /dev/nvme0n1p5 /tmp/root-b
cat /tmp/root-a/etc/lsb-release | grep -iE chromeos_release_\(build\|version\)
cat /tmp/root-b/etc/lsb-release | grep -iE chromeos_release_\(build\|version\)
```
Pick the partition with the later release (higher numbers) and run
```
sudo dd if=/dev/nvme0n1p3 of=/home/cros/cros/Download/cros.img status=progress
# Change nvme0n1p3 to nvme0n1p5 if root-B has a higher version
```
This will save a raw image copy of the root filesystem to your Download folder, from which you can copy it to an external media. Before you flash BIOS.

After you have installed MrChromeBox's custom firmware and then Linux (hopefully Debian 11\*, and you should find at least keyboard and mousepad work well, screen's a tad too bright, and there's no sound). Install the necessary packages to build the kernel: 
```
sudo apt install git make automake autoconf bc libncurses-dev libtool build-essential flex bison
```
\* If you want to use wifi during the installation for Debian 10/11, you will need to use the installation media with non-free firmware included (Ubuntu and Gallium OS installer already include these). Otherwise the installer will prompt you to insert a USB thumbdrive with the firmware saved in a `.deb` package (available in repository as `firmware-linux-nonfree` and `firmware-linux-misc`), but your USB ports will be wonky at this point and may not read an additional thumbdrive. In any case, a powered hub for the installation media is recommended.

Make a note of the linux version in your Chrome OS root filesystem image (`CHROME_IMG_PATH=/path/to/cros.img` you created earlier), Chrome Recovery Media (`CHROME_MEDIA_PATH=/dev/sdn`), or Chrome Recovery Image (`CHROME_RECOVERY_PATH=/path/to/some_file_name.bin`). Whichever it be, mount it and extract a few things:
```
# Mount the Chrome OS root filesystem
sudo mount -t ext2 -o ro $CHROME_IMG_PATH /mnt
# OR mount the Chrome Recovery Media
sudo mount -t ext2 -o ro $CHROME_MEDIA_PATH /mnt
# OR mount the Chrome Recovery Image
sudo losetup /dev/loop99 $CHROME_RECOVERY_PATH
sudo partprobe /dev/loop99
sudo mount -t ext2 -o ro /dev/loop99p3

# Copy the following files to your Linux filesystem
sudo cp -r /mnt/opt/* /opt
sudo cp -r /mnt/usr/lib/firmware/* /lib/firmware

# Note or copy these folders and files for later use:
cd; mkdir cros-files
cp -r /mnt/lib/udev/rules.d ~/cros-files/udev-rules.d
cp -r /mnt/usr/share/alsa/ucm ~/cros-files/alsa-ucm

# ... and copy the following file
cp /mnt/lib/modules/*/kernel/kernel/configs.ko ~/cros-files/configs.ko
```
### Getting Your Pixelbook Ready
### Installation

## Tips
### Custom Kerynel Configuration
* If you want to make your own kernel, the custom kernal configuration provided here is a good start point. It can certainly be made leaner, and there are always more modules to add to suit your needs.
* At minimum, your kernel should contain everything that's in the Chrome OS kernel, with the exception that disable PCI hotplug must be disabled, or Linux sometimes won't even detect the wifi adapter at boot and when waking from suspend.
* It also takes more than what's on the Chrome OS kernel to run Linux, since many hardware functions on Chrome OS are driven by userspace applications not readily available on any Linux distro, and you'll need to enable kernel driver modules for these devices to properly function.
### Custom Key Mapping
* `evtest` will return the needed key event on the builtin keyboard needed for configuring custom keyboard rules.
### Dual Boot Windows 10
* The same rule regarding using a powered hub applies unless you have upgraded EC firmware.
* Depending on how you obtained your Windows 10 installation media, there's a good chance that it won't recognize the internal NVME harddrive and will ask you for a driver. If this is the case and you're not able to secure a version that supports EVE's internal drive, you can reserve a partition on the internal harddrive, pass the entire /dev/nvme0n1 to a VM, install Windows inside the VM on that partition (be sure to select UEFI instead of BIOS but do not select the secure boot image; you can even share EFI partition with Linux by passing the whole block device), then reboot natively into Windows. Note that once you boot natively into Windows, you will no longer be able to run it from the VM.
* With the latest Windows updates, Windows 10 runs pretty smooth natively and has driver for all the hardware components. Just make sure that you get all the updates and hold off on the feature upgrades until you get all the drivers. This may take an hour or so, and you may need to go into Device Manager to rush things along as some devices may not get their driver automatically.
* Try Windows 10 in a virtual machine!
### Rescue Partition
* Given the finicky USB, it might be a good idea to install another lightweight Linux with with utilities like `gparted` and `chroot` on a small partition in case you need to do rescue work on the go. CloneZilla, which is based on Debian, is just a couple hundred MBs and has a GUI. Even a minimal install of Ubuntu with default GNOME desktop won't take more than 10G.
### Troubleshooting
The following tools are useful for troubleshooting
* `dmesg -T`
* `journalctl`
* `acpi_listen`