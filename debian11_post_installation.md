# Debian 11 XFCE post installation

PC: Lenovo YOGA Slim 7 14ARE05 (AMD Ryzen 7, AMD Radeon Graphics)

## Packages:

```
sudo apt install firmware-linux-free
sudo apt install firmware-iwlwifi
```
Optional:

```
sudo apt install firmware-linux
```
(non-free/metapackages)
Needs the following or similar line in ```/etc/apt/sources.list``` or e.g.```/etc/apt/sources.list.d/debain-non-free.list```:

```deb http://ftp.no.debian.org/debian bullseye main non-free```

## Environment variables:

Create ```~/.xsessionrc```,
and add the following lines:

```
export XDG_CONFIG_HOME="$HOME/.config
export XDG_DATA_HOME="$HOME/.local/share"
export PATH="$PATH:~/.local/bin"
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
and add UUID and offset of swapfile (uncomment ```GRUB_CMDLINE_LINUX_DEFUALT```):

Example:

```GRUB_CMDLINE_LINUX="quiet splash resume=UUID=dc64f684-08da-4f47-9180-5923edfd5fbf resume_offset=6422528"```

Run ```sudo update-grub```

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

Provides graphical UI to bluez, includes system tray applet.

## Bugs:

bluetootd will report error on startup regarding ```sap```:
e.g. "profiles/sap/server.c:sap_server_register() Sap driver initialization failed."
and "sap-server: Operation not permitted (1)"
SAP is for old equipment and can be ignored. Ref. https://unix.stackexchange.com/questions/705326/debian-11-bluetooth-sap-driver-initialization-failed
To stop messages, edit the bluetoothd.service file and add the following flag```--noplugin=sap``` in exec-line:
```ExecStart=/usr/libexec/bluetooth/bluetoothd --noplugin=sap```
