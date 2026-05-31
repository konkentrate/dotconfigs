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
