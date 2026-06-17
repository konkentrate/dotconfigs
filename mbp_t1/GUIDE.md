# Preface

All of the below were done using the latest version of CachyOS as of 31.05.2026. Other kernels might differ.

# WiFi Fixes
For BCM43602 chipset:
```
sudo nano /etc/default/grub
```
Add this inside GRUB_CMDLINE_LINUX_DEFAULT:
```
GRUB_CMDLINE_LINUX_DEFAULT='quiet splash  brcmfmac.feature_disable=0x82000'
```
Update GRUB:
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
## Disable Wi-Fi Power Saving
```
sudo nano /etc/NetworkManager/conf.d/wifi-powersave-off.conf
```
Paste:
```
[connection]
wifi.powersave=2
```
Reboot, you can verify it worked with: `iw dev wlan0 get power_save`

## 2.4/5 GHZ band fix
```
sudo nano /usr/lib/firmware/brcm/brcmfmac43602-pcie.txt
```
Paste in content from the .txt file, and replace your MAC address (find using `ip addr`) of your Wi-Fi card. Then run:
```
sudo rmmod brcmfmac_wcc && sudo rmmod brcmfmac && sudo modprobe brcmfmac
```

# Audio

Install drivers from https://github.com/davidjo/snd_hda_macbookpro

# USB-C fix

For some reason, USB 3.0 adapters were not working for me. I fixed this by adding the following parameters to the GRUB configuration, inside GRUB_CMDLINE_LINUX_DEFAULT:

```
intel_iommu=on iommu=pt pcie_ports=compat
```
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

After rebooting, the Touch Bar stopped working again for some reason.
Running: sudo mkinitcpio -P and rebooting fixed it.

# Touchbar

```
sudo git clone https://github.com/jimmykuo/macbook12-spi-driver.git /usr/src/applespi-0.1
```
```
sudo nano /etc/mkinitcpio.conf
```

and paste:
```
MODULES=(applespi spi_pxa2xx_platform intel_lpss_pci apple_ib_tb apple_ibridge apple_ib_als)
```
Install:
```
sudo dkms install -m applespi -v 0.1
sudo mkinitcpio -P
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

Then reboot.

## Invert FN Keys:
```
sudo mkdir -p /etc/modprobe.d
```
```
echo "options apple_ib_tb fnmode=2" | sudo tee /etc/modprobe.d/apple-ib-tb.conf
```
```
sudo mkinitcpio -P
```
Then reboot.

<<<<<<< HEAD
# MacBook Pro Touchbar Fix After Sleep (Linux)

## Problem

On Linux with the `applespi` drivers, the touchbar goes blank after waking from s2idle suspend and doesn't come back until a full reboot.

## Root Cause

The touchbar is a USB HID device (`05ac:8600`, Apple iBridge) connected internally via USB. After s2idle suspend, the USB device doesn't reinitialize properly on wake. Simply reloading the kernel modules (`apple_ib_tb`, `apple_ibridge`) is not enough — the USB device itself needs to be reset.

## Fix

Create a systemd sleep hook that resets the USB device on resume.

### 1. Find the USB device path

```bash
lsusb | grep -i apple
# Bus 001 Device 002: ID 05ac:8600 Apple, Inc. iBridge

# Find the sysfs path
cat /sys/bus/usb/devices/1-3/idVendor
# Should output: 05ac
```

The device is at `/sys/bus/usb/devices/1-3`.

### 2. Create the sleep hook

Scripts must go in `/usr/lib/systemd/system-sleep/` (not `/etc/systemd/system-sleep/`).

```bash
sudo nano /usr/lib/systemd/system-sleep/touchbar-resume.sh
```

```bash
#!/bin/bash
if [ "$1" = "post" ]; then
    sleep 2
    echo 0 > /sys/bus/usb/devices/1-3/authorized
    sleep 1
    echo 1 > /sys/bus/usb/devices/1-3/authorized
fi
```

```bash
sudo chmod +x /usr/lib/systemd/system-sleep/touchbar-resume.sh
```

### 3. Test manually

systemd calls sleep hooks with two arguments, so test like this:

```bash
sudo /usr/lib/systemd/system-sleep/touchbar-resume.sh post suspend
```

The touchbar should flicker and come back within ~3 seconds.

### 4. Test on real suspend

```bash
sudo systemctl suspend
```

On wake the touchbar should restore automatically.

## Bonus: Fix slow wake (65 second delay)

The Thunderbolt 3 USB controllers (`JHL6540`) were timing out on resume causing ~65 second wake times. Fix with a udev rule:

```bash
sudo nano /etc/udev/rules.d/99-thunderbolt-resume.rules
```

```
ACTION=="add", SUBSYSTEM=="pci", ATTR{vendor}=="0x8086", ATTR{device}=="0x15d4", ATTR{power/control}="on"
```

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

Wake time dropped from ~65 seconds to ~1 second.
=======
# Sleep / Suspend

The fix here should be done so the laptop wakes at all: https://github.com/Dunedan/mbp-2016-linux#suspend--hibernation

Optionally for faster wake up (but more battery consumption) set up S2IDLE instead of DEEP sleep. Add this to your /etc/default/grub GRUB_CMDLINE_LINUX_DEFAULT
```
mem_sleep_default=s2idle
```
>>>>>>> 451fc1f1330b86b6a158fd84ed41eb8a2fb99014
