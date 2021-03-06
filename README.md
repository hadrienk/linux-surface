# Linux Surface

Linux running on the Surface Book, Surface Book 2, Surface Pro 3, Surface Pro 4, Surface Pro 2017 and Surface Laptop. Follow the instructions below to install the latest kernel and config files.

### What's Working

* Keyboard (and backlight) (not yet working for Surface Laptop)
* Touchpad
* 2D/3D Acceleration
* Touchscreen
* Pen
* WiFi
* Bluetooth
* Speakers
* Power Button (not yet working for SB2/SP2017)
* Volume Buttons (not yet working for SB2/SP2017)
* SD Card Reader
* Cameras (partial support, disabled for now)
* Hibernate
* Sensors (accelerometer, gyroscope, ambient light sensor)
* Battery Readings (not yet working for SB2/SP2017)
* Docking/Undocking Tablet and Keyboard
* Surface Docks
* DisplayPort
* USB-C (including for HDMI Out)
* Dedicated Nvidia GPU (Surface Book 2)

### What's NOT Working

* Dedicated Nvidia GPU (if you have a performance base on a Surface Book 1, otherwise onboard works fine)
* Cameras (not fully supported yet)
* Suspend (uses Connected Standby which is not supported yet)

### Disclaimer
* For the most part, things are tested on a Surface Book. While most things are reportedly fully working on other devices, your mileage may vary. Please look at the issues list for possible exceptions.

### Download Pre-built Kernel and Headers

Downloads for ubuntu based distros (other distros will need to compile from source using the included patches):
https://github.com/jakeday/linux-surface/releases

Downloads for arch based distros:
https://github.com/pharra/linux-surface/releases

You will need to download both the image, headers and libc-dev files for the version you want to install.

### Instructions

Surface Series Devices:
* Series 5 devices: Surface Book 2, Surface Pro 2017
* Series 4 devices: Surface Book, Surface Pro 4, Surface Laptop
* Series 3 devices: Surface Pro 3

For the ipts_firmware files (series 4/5 devices only), please select the version for your device.
* v76 for the Surface Book
* v78 for the Surface Pro 4
* v79 for the Surface Laptop
* v101 for Surface Book 2 15"
* v102 for the Surface Pro 2017
* v137 for the Surface Book 2 13"

For the i915_firmware files (series 3/4/5 devices), please select the version for your device.
* kbl for series 5 devices
* skl for series 4 devices
* bxt for series 3 devices

These steps assume are you in the linux-surface repo.

1. Copy the files under root to where they belong:
  ```
   sudo cp -R root/* /
  ```
2. Make /lib/systemd/system-sleep/hibernate as executable:
  ```
   sudo chmod a+x /lib/systemd/system-sleep/hibernate
  ```
3. (Series 4/5 only) Extract ipts_firmware_[VERSION].zip to /lib/firmware/intel/ipts/
  ```
   sudo mkdir -p /lib/firmware/intel/ipts
   sudo unzip firmware/ipts_firmware_[VERSION].zip -d /lib/firmware/intel/ipts/
  ```
4. (Series 3/4/5 only) Extract i915_firmware_[VERSION].zip to /lib/firmware/i915/
  ```
  sudo mkdir -p /lib/firmware/i915
  sudo unzip firmware/i915_firmware_[VERSION].zip -d /lib/firmware/i915/
  ```
5. (Surface Book 2 only) Extract nvidia_firmware_gp108.zip to /lib/firmware/nvidia/gp108/
  ```
  sudo mkdir -p /lib/firmware/nvidia/gp108
  sudo unzip firmware/nvidia_firmware_gp108.zip -d /lib/firmware/nvidia/gp108/
  ```

6. (Ubuntu 17.10) Fix issue with Suspend to Disk:
  ```
  sudo ln -s /lib/systemd/system/hibernate.target /etc/systemd/system/suspend.target && sudo ln -s /lib/systemd/system/systemd-hibernate.service /etc/systemd/system/systemd-suspend.service
  ```
7. (all other distros) Fix issue with Suspend to Disk:
  ```
  sudo ln -s /usr/lib/systemd/system/hibernate.target /etc/systemd/system/suspend.target && sudo ln -s /usr/lib/systemd/system/systemd-hibernate.service /etc/systemd/system/systemd-suspend.service
  ```
8. Install the latest marvell firmware (if their repo is down, use my copy at https://github.com/jakeday/mwifiex-firmware):
  ```
  git clone git://git.marvell.com/mwifiex-firmware.git
  sudo mkdir -p /lib/firmware/mrvl/
  sudo cp mwifiex-firmware/mrvl/* /lib/firmware/mrvl/
  ```
9. Install the headers, kernel and libc-dev (or follow the steps for compiling the kernel from source below):
  ```
  sudo dpkg -i linux-headers-[VERSION].deb linux-image-[VERSION].deb linux-libc-dev-[VERSION].deb
  ```
10. Reboot on installed kernel.

### Compiling the Kernel from Source

If you don't want to use the pre-built kernel and headers, you can compile the kernel yourself following these steps:

0. (Prep) Install the required packages for compiling the kernel:
  ```
  sudo apt-get install build-essential binutils-dev libncurses5-dev libssl-dev ccache bison flex
  ```
1. Assuming you cloned the linux-surface repo (this one) into ~/linux-surface, go to the parent directory:
  ```
  cd ~
  ```
2. Clone the mainline stable kernel repo:
  ```
  git clone git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git
  ```
3. Go into the linux-stable directory:
  ```
  cd linux-stable
  ```
4. Checkout the version of the kernel you wish to target (replacing with your target version):
  ```
  git checkout v4.y.z
  ```
5. Apply the kernel patches from the linux-surface repo (this one):
  ```
  for i in ~/linux-surface/patches/[VERSION]/*.patch; do patch -p1 < $i; done
  ```
6. Get current config file and patch it:
  ```
  cat /boot/config-$(uname -r) > .config
  patch -p1 < ~/linux-surface/patches/config.patch
  ```
7. Compile the kernel and headers (for ubuntu, refer to the build guild for your distro):
  ```
  make -j \`getconf _NPROCESSORS_ONLN\` deb-pkg LOCALVERSION=-linux-surface
  ```
8. Install the headers, kernel and libc-dev:
  ```
  sudo dpkg -i linux-headers-[VERSION].deb linux-image-[VERSION].deb linux-libc-dev-[VERSION].deb
  ```


### Build in archlinux build system

It will build the packages for archlinux
Forked from jakeday/linux-surface, I have tested it in my surface pro4. 
```
git clone https://github.com/jakeday/linux-surface.git
cd linux-surface/build
makepkg -si
```

### NOTES

* If you are getting stuck at boot when loading the ramdisk, you need to install the Processor Microcode Firmware for Intel CPUs (usually found under Additional Drivers in Software and Updates).
* If you are having issues with the position of the cursor matching the pen/stylus, you'll need to update your libwacom as mentioned here: https://github.com/jakeday/linux-surface/issues/46

### Donations Appreciated!

PayPal: https://www.paypal.me/jakeday42

Bitcoin: 1AH7ByeJBjMoAwsgi9oeNvVLmZHvGoQg68
