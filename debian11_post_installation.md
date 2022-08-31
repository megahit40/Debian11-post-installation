# Debian 11 XFCE post installation

PC: Lenovo YOGA Slim 7 14ARE05 (AMD Ryzen 7, AMD Radeon Graphics)

## Packages:

```
sudo apt install firmware-linux-free
sudo apt install firmware-iwlwifi
```

## Environment variables:

Create ```~/.xsessionrc```
and add the following lines:

```
export XDG_CONFIG_HOME="$HOME/.config
export XDG_DATA_HOME="$HOME/.local/share"
export PATH=$PATH:"~/.local/bin"
export PATH="$PATH:~/.scripts"
```

## Swapfile (assuming no swap partition of course):

```
sudo fallocate -l 17G /swapfile
```
or
```
sudo dd if=/dev/zero of=/swapfile bs=xxxx count=xxxxx
```

```
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

edit ```/etc/fstab```,
and add the following line:

```
/swapfile swap swap defaults 0 0
```

Verify:

```
sudo swapon --show
```

(To remove swapping:

```
sudo swapoff -v /swapfile
sudo rm /swapfile
```
(and delete ```/swapfile swap swap defaults 0 0``` from ```/etc/fstab```)

## Hibernation:

### Suspend, then hibernate:

Edit /etc/systemd/logind.conf:

```
HandleLidSwitch=suspend-then-hibernate
```

Check ```/etc/systemd/sleep.conf```:

```
AllowSuspendThenHibernate=yes
```

### Find UUID and offset of ```/swapfile```:

Physical offset:

```
sudo filefrag -v /swapfile
```

UUID:

```
sudo findmnt -no UUID -T /swapfile
```

Edit ```/etc/default/grub```,
find ```GRUB_CMDLINE_LINUX```,
and add UUID and offset of swapfile:

Example:

```GRUB_CMDLINE_LINUX="resume=UUID=dc64f684-08da-4f47-9180-5923edfd5fbf resume_offset=6422528"```

#### xfce4-power-manager will by default override logind.conf.

Query status of Xfce power manager:

```
xfce4-power-manager --dump
```

Query handling of logind-handle-lid-switch:

```
xfconf-query -c xfce4-power-manager -p /xfce4-power-manager/logind-handle-lid-switch
```

If ```false```, change to ```true``` with the following command:

```
xfconf-query -c xfce4-power-manager -p /xfce4-power-manager/logind-handle-lid-switch -n -t bool -s true
```

Verify with:

```
xfconf-query -c xfce4-power-manager -p /xfce4-power-manager/logind-handle-lid-switch
```

## Screen tearing AMDGPU:

Create ```/etc/X11/xorg.conf.d/20-amdgpu.conf```

and add the following to it:

```
Section "Device"
  Identifier "AMD Graphics"
  Driver "amdgpu"
  Option "TearFree" "true"
EndSection
```

## Bluetooth

```sudo apt install blueman```

Provides graphical UI to bluez, includes system tray applet
