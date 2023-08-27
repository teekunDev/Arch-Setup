# My Arch Setup
This is my current Arch-Setup and because I have alzheimers I am documenting it here including all the Conig-Files for Hyprland, Waybar and such.

I tried to keep this guide as simple and easy to understand as possible

This README also includes the whole Arch-Install

>   I would recommend you making a completly clean Arch install with this guide so you won't have any Problems \
    If you are sure that you know what you are doing you can skip the Setup part and install the dependencies.

<div align="left">
<details>
<summary>Dependencies</summary>

- ``sudo``
- ``nano``
- ``git``
- ``intel-ucode`` or ``amd-ucode`` depending on your processor.
- ``networkmanager``
- ``neofetch``
- ``btop``
- ``base-devel``
- ``linux-lts``
- ``linux-headers``


</details>
<p></p>
</div>


**Requirements**
- A USB-Stick with the Arch-Iso
- A AMD GPU (NVIDA GPUs need a diffrent way to install Hyprland see [Wiki](https://wiki.hyprland.org/Nvidia/))
- Empty Volume on Disk for the partitions
- BIOS in UEFI Mode and Secure Boot off

<div>
<details>
<summary><b>Arch install</b> </summary>

## Arch Install

### Check if you booted in UEFI-Mode

You can easily check with the following command

```bash
$ ls /sys/firmware/efi/efivars
```

If u can see tons of text than u can be sure you are in UEFI-Mode

### Make sure you are connected to the Internet

```bash
$ ping google.com
```

If you're getting a Response from ``google.com`` than you can be sure that you are connected to the internet

### Set Timezone

First of all check if the time is already set right

```bash
$ timedatectl status
```

If ``Local time`` isn't right we need to set the right timezone

First get the right timezone with ``list-timezones``

```bash
$ timedatectl list-timezones
```

Now run ``set-timezone`` with the timezone you want for example ``Europe/Vienna`` to set the timezone

```bash
$ timedatectl set-timezone Europe/Vienna
```

If you run ``status`` again it should give back the right time


### Load the right Keyboard 

**If you are fine with the regular ``en`` Layout then skip this**

First list all the avaiable Keyboard layouts

```bash
$ localectl list-keymaps
```
Now run ``loadkeys`` with your keyboard layout for example ``de-latin1``

```bash
$ loadkeys de-latin1
```


### Partition Table

The partitioning Process is pretty straight forward

U basically only need 3 Partitions
- A ``EFI`` Partition (At least 100M) \
    If you have a Windows install and want to dual boot, you won't need to make one.
- A ``root`` Partition (with at least 32G)
- A ``swap`` Partition (with atleast half capacity of your memory)
- A ``home`` Partition (can have the rest)

**How to partition your drive you can lookup on the [Wiki](https://wiki.archlinux.org/title/partitioning) if you don't know how to \
I recommend using ``cfdisk``**

### Formating

Formating is also straigt forward. 

To Format your ``root`` and ``home`` Partitions just run the following

```bash
$ mkfs.ext4 /dev/root_partition
$ mkfs.ext4 /dev/home_partition
```

For the ``swap`` partition do

```bash
$ mkswap /dev/swap_partition
$ swapon /dev/swap_partition
```

If you have a new/seperate ``EFI`` partition do
```bash
$ mkfs.fat -F 32 /dev/efi_partition
```

### Setting the Label/Mountpoints

This is also very straight forward.

For the ``root`` partition run

```bash
$ mount /dev/root_partition /mnt
```

For the ``home`` partition do

```bash
$ mkdir /mnt/home
$ mount /dev/home_partition /mnt/home
```

For the ``EFI`` partition do

```bash
$ mkdir /mnt/boot/EFI
$ mount /dev/efi_partition /mnt/boot/EFI
```

### Pacstrap

Now you are going to install Arch to the root partition with ``pacstrap -i /mnt``

We can now also decide what we wanna pre-install. \
Most importantly is:
- ``base``
- ``linux``
- ``linux-firmware``

U need to install these to get Arch even working

Now we also add the packages I like to pre-install
- ``sudo``
- ``nano``
- ``git``
- ``intel-ucode`` or ``amd-ucode`` depending on your processor.
- ``networkmanager``
- ``dhcpcd``
- ``neofetch``
- ``btop``
- ``base-devel``
- ``linux-lts``
- ``linux-headers``

So for me it would look like this

```bash
$ pacstrap -i /mnt base linux linux-firmware sudo nano amd-ucode networkmanager neofetch btop base-devel linux-lts linux-headers dhcpcd
```


### Generate the File System Table

Generating the File System Table is also straight forward

Just run this
```bash
$ genfstab -U /mnt >> /mnt/etc/fstab
```

### CHRoot

CHRoot will let us go into the Arch now

This is also straight forward
```bash
$ arch-chroot
```

### Set Keyboard Layout

If you don't remember your Keyboard Layout run

```base
$ localectl list-keymaps
```

Then run this with your *Keymap* for me it's still ``de-latin1``

```base
$ localectl set-keymaps --no-convert de-latin1
```

### Change Root Password

Run this

```bash
$ passwd
```

### Add Standard User

Create the User with ``useradd -m``

```base
$ useradd -m [username]
```

Change the password with ``passwd``

```bash
$ passwd [username]
```

Give the User the standard rights with ``usermod``

```bash
$ usermod -aG wheel,storage,power [username]
```

To finalize it you have to change some stuff in the ``sudo`` config

Run the following with your desired text editor

```bash
$ EDITOR=[text_editor] visudo
```


Uncomment the following line
```bash
#%wheel ALL=(ALL) ALL
```

And then add the line ``Defaults timestamp_timeout=[time]``

The result should look like this
```bash
%wheel ALL=(ALL) ALL
Defaults timestamp_timeout=[time]
```
Save and exit


### Set System Language

Open ``/etc/locale.gen`` with your text editor and uncomment the desired language and save it

After that run this
```bash
$ locale-gen
$ echo LANG=[language] >  /etc/locale.conf 
$ export LANG=[language]
```

### Setting up timezone/region

For this u need to get the timezone itself

Run the following 
```bash
$ ln -sf /usr/share/zoneinfo/[Continent]/[City] /etc/localtime
$ hwclock  --systohc
```

### Setup Hostname

Give Arch a hostname through running this
```bash
$ echo [hostname] > /etc/hostname
```

Then add the following to  ``/etc/hosts``

```bash
127.0.0.1   localhost
::1         localhost
127.0.1.1   [hostname].localdomain      localhost
```

After saving it run the following

```bash
$ systemctl enable dhcpcd.service
$ systemctl enable NetworkManager.service
```

### GRUB Install

You now need a Bootloader. \
Because I like it we are going to use GRUB

First of all you have to install ``GRUB``, ``efibootmgr``, ``dosfstools`` and ``mtools`` by running.
```bash
$ pacman -S grub efibootmgr dosfstools mtools
```


There are 2 options to install the bootloader

<div>
<details>
<summary><B>Dual-Boot/Multiboot</B></summary> 

For the Multiboot you will also need ``os-prober``

```bash
$ pacman -S os-prober
```

Then u need to uncomment the following line in ``/etc/default/grub``

```bash
#GRUB_DISABLE_OS_PROBER=false
```
After saving it u need to install GRUB with ``grub-install`` and create the config with ``grub-mkconfig``

```bash
$ grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck
$ grub-mkconfig -o /boot/grub/grub.cfg
```

GRUB is now installed

> If it didn't detect your Windows Bootloader run the command again once you are in Hyprland

<p></p>

</details>
<details>
<summary><B>Singleboot</B></summary>
If you Singleboot you just need to install GURB with ``grub-install`` and ``grub-mkconfig``

```bash
$ grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck
$ grub-mkconfig -o /boot/grub/grub.cfg
```

GRUB is now installed

</details>
</div>

### Finalizing

Exit Arch with ``exit``

Now Unmount your Arch ``root`` partition and reboot your system
```bash
$ umount -lR /mnt
$ reboot 
```

> Don't forget to set the Bootloader in your BIOS to ``grub_uefi`` otherwise it will start your old bootloader
</details>
</div>

## **yay**

A helper for handling AUR Packages

```bash
$ sudo pacman -S --needed git base-devel
$ git clone https://aur.archlinux.org/yay.git
$ cd yay
$ makepkg -si
``` 

## Hyprland 

My chosen Wayland Compositor. It's just so goddamn cool

The install itself is very straight forward


```bash
$ yay -S hyprland-git waybar-hyprland-git hyprpaper rofi dunst rofi-power-menu
```

After that just Press ``ENTER`` and ``y`` when it promts something to use the defaults

Now you also need to install ``kitty``

```bash
$ sudo pacman -S kitty
```
>   Any other Terminal will do it's job but for my Hyprland config ``kitty`` is needed. \
    This can be changed in the config

## Base Configuration

<div>
<details>
<summary><b>Dependencies</b></summary>

```bash
## Fonts and Theme
$ yay -S ttf-font-awesome ttf-icomoon-feather ttf-jetbrains-mono ttf-nerd-fonts-symbols noto-fonts noto-fonts-cjk noto-fonts-emoji papirus-icon-theme catppuccin-gtk-theme-mocha 
```

```bash
## Audio and Video
$ yay -S pipewire-alsa pipewire-jack pipewire-pulse xdg-desktop-portal-hyprland-git pamixer pavucontrol wireplumber grim slurp tumbler
```

> You may need to uninstall any other `xdg-desktop-portal` Package that isn't `xdg-desktop-portal-gtk` and `xdg-desktop-portal-hyperland-git` just check with `pacman -Q | grep xdg-desktop-portal-`

```bash
## Network
$ yay -S nm-connection-editor dhcpcd networkmanager
```

</details>
</div>

<div>
<details>
<summary><b>Apps and others</b></summary>

```bash
## File Explorer
$ yay -S thunar thunar-archive-plugin file-roller gvfs ffmpegthumbnailer
```

```bash
## Browser
$ pacman -S firefox
```

```bash
## Discord
$ yay -S webcord
```

```bash
## VS Code
$ yay -S code code-features code-marketplace
```

```bash
## Remote stuff
$ yay -S remmina openvpn networkmanager-openvpn freerdp
```

<div>
<details>
<summary>Steam</summary>

First if not already you have to uncomment `[multilib]` in the Pacman config

```bash
$ sudo {editor} /etc/pacman.conf
```

```bash
#[multilib]
#Include = /etc/pacman.d/mirrorlist
```

Update the pacman databases and might aswell upgrade the packages

```bash
sudo pacman -Syu
```

Once finished u can install Steam

```bash
$ sudo pacman -S steam
```

> Don't forget to activate Proton

</details>
</div>

<p></p>

</details>
</div>

<div>
<details>
<summary><b>DOTFILES</b></summary>

> If you don't like everything just cherry pick the things you like just don't forget to copy the whole folder for the scripts

> I am to lazy to write all the Keybinds down. just check the `.config/hypr/keybinds.conf`

Clone the repo and after that use rsync to get everything 

```bash
sudo pacman -S rsync
```

```bash
$ git clone https://github.com/teekunDev/Arch-Setup.git ~/Dotfiles
rsync ~/Dotfiles ~/
```

After that do a full reboot

Now you are good to go

</details>
</div>

## Credits

© _**Artist who make Wallpapers, graphics and more**_

© _**[linuxmobile](https://github.com/linuxmobile/hyprland-dots) whose project this is based of**_
