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
Screen Backlight | Not adjustable | Custom kernel | Adjustable backlight through desktop builtin control
Touchpad | Not functioning or awkward | Custom kernel | Fully functional
Wifi | Failture to detect device at boot or wakeup | Custom kernel | Fully functional
Sound | No sound, poor control, peripherals not recognized | Two approaches | Fully functioning, customizable sound with basic and auto controls
USB | Not recognizing devices, shoddy connection | Firmware update | Fully functioning USB ports as experienced with ChromeOS
Keyboard | Lacking certain keys | Custom key mapping | Replacing certain keys with Super, Caplock, PageUp, PageDown, Home, End, and Delete

The kernel configuration in the default setup additionally meets the the kernel requirements for a host of common applications while keeping the kernel relatively free of bloat.

The following virtualization/containerization platforms are supported:
* `libvirt-qemu`
* `lxd`
> Note: network filtering and filesystem support is minimal, additional kernel options may need to be enabled for certain virtualization and firewall configurations.

## Outstanding issues
* **USB fix is experimental with watchdog disabled in the custom firmware; this may potentially cause issues down the line, and has a noticeable power consumption impact likely due to a high amount of IRQs.**
* Laptop display rotated 180 degrees in some desktop environment, multiple display setup, and tablet mode, likely due to miscommunication with sensors (accelerometer, gyroscope, and/or lid angle). There are a couple easy fixes for errant screen rotation in desktop mode if the presents (see section), 
* Some kernel error messages with no known or significant functional impact. A couple at boot, and a few recurring ones related to sound.

## Compatibility and requirements

Files contained herein are developed in and for the following environment:
* Hardware: Pixelbook i-7 (eve)
* Kernel:   Linux 5.4.104 (Chromium R90)
* OS:       Debian 11 (`bullseye`)
* Desktop:  GNOME 3.38

It is my hope that this git can inform Chromebook Linux enthusiasts in any way big or small regardless of their hardware and distribution choices, and while these are not hard requirements, they are the recommended setup to fully take advantage of this git. Compatibility of this git on your device and distro will vary, as discussed below:

### Hardware

A lot of issues and subsequent fixes with Linux on Pixelbook has to do with the peculiarities of eve hardware, which lacks conventioanl Linux support. If you're using a different Chromebook, chances are that it is already better supported by common Linux distros. If you're using a Pixelbook, i-3 and i-5 flavors may not have enough computing power to run GNOME smoothly, which can be a resource hog, though this git is desktop-environment-agnostic. If you're using a Pixelbook Go, the configurations here won't work properly without modifications, though you may still benefit from revewing them for a jumpstart.

### Kernel

Certain things just won't work without using the Chromium-flavored Linux kernel, including screen backlight and sound. Kernel v5.4 has been used by ChromeOS on Pixelbook since R89 (v4.4 prior to that). Kernel v4.4 works well with Debian 10/Ubuntu/Gallium, but lacks GPU acceleration in Debian 11 (likely due to a `mesa` bug), hence kernel v5.4 is recommended.

### OS

Debian 11 This git should work with few to no modification on Debian-based operating systems that use `systemd` and `apt`, which include Ubuntu and Gallium OS (which is based on Ubuntu), and most likely will <b>not</b> work outside of these parimeters. 

Debian 11 `bullseye` is the tested OS for this git and it works great out of box on Pixelbook i-7. Compared to Debian 10 `buster`, Ubuntu (both 18.04LTS and 20.04LTS), and even GalliumOS, `bullseye` has excellent support for eve touchpad, keyboard backlight, and HiDpi display (thanks to an upgrade to GNOME 3.38). However, like with the other distros and as noted above, without the Chromium-flavored kernel and proper configuration, several key interfaces do not work properly, most notably sound, screen backlight, and USB ports. 

## Optional preparation

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
## Usage
To install the latest configurations and utility scripts, simply run from a writeable directory of your choice
```
git clone git://github.com/seamus-x/linux-on-eve
sudo make all
```
This will perform the following tasks:
1. Download Linux kernel and firmware for Chromium OS along with required dependencies for kernel building, build the kernel and modules according to this git's default for kernel release and configuration file, and install to your machine.
    * For this step only, run `sudo make kernel`
    * Set `CROS_KERNEL_VERSION` to override the kernel release version (performs a `git switch` so make sure it is set to an extant branch);
    * Set `CROS_KERNEL_CONFIG` to override the kernel configuration file (uses `make olddefconfig`).
2. Download the Chrome OS Recovery Image and extracts configuration files.
    * To avoid the inclusion of non-free code in this git, this step cannot be skipped unless:
        * Making task 1 only;
        * Your own recovery image, media, or root filesystem image is supplied (choose one only; only one will be used if more than one override is given) by setting the following environmental variables:
            * `CROS_IMAGE_OVERRIDE=/path/to/img`;
            * `CROS_MEDIA_OVERRIDE=/dev/path`;
            * `CROS_ROOTFS_OVERRIDE=/path/to/img`.
        * Your own configuration files are supplied for the specific task. See tasks below for details.
3. Set up key mapping and install associated dependencies.
    * Linux recognizes the top-row Chrome keys as F1-F10, F13. The default script will remap the following keys:
        * Google Assistant Key -> Super L
        * Search Key -> Caplock
        * Hamberger Key -> F11
        * Shift R -> Delete
        * Alt R -> Home
        * Ctrl R -> End
        * Volume Up (Side) -> PageUp
        * VOlume Down (Side) -> PageDown
    * For step 2 and this step only, run `sudo make keymapping`;
    * To override default keymapping and to skip step 2, set `KEYMAPPING_CONFIG_OVERRIDE=/path/to/keymapping-config.rules`.
4. Set up swap
    * To use zram, `CONFIG_ZRAM` must be enabled in the kernel.
    * To use a disk-based swap file, `CONFIG_DISK_BASED_SWAPFILE` must be enabled in the kernel.
    * To 

## Tips
### 

### Custom Kerynel Configuration

### Custom Key Mapping
`evtest` will return the needed key event to configure custom keyboard rules.
### 
### Troubleshooting
The following tools are useful for troubleshooting
* `dmesg -T`
* `journalctl -xe`
* `cras_test_client`
* `acpi_listen`