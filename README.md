# Arch Install Guide

### Basic Guideline for ANY linux distro

1. [Create a live ISO onto a disk](#1-live-iso)
    - [Steps for dual booting windows](#15-setting-up-windows-for-dual-booting)
2. [Setup the BIOS/UEFI for linux](#2-biosuefi-setup)
3. [Boot into the live ISO](#3-boot-into-live-iso)
4. [Use the live ISO to write a linux install into the main disk](#4-installing-on-live-iso)
5. [Reboot into the main install and finish the setup](#5-actual-install)
6. [Install any and all applications and setting up the system for your liking](#6-other-things)

This guide will be going through ARCH linux specifically. For any other distro, step 4 and 5 are very simple and just need to follow the ISO's guide.

**NOTE:** In this guide, I will be using \<Text\> to be placeholders. Replace them without the chevrons (Angle brackets).

**NOTE:** It is preferable not to leave the setup between chapters (Except for 6). Some steps will have notes for repeating them if you leave within a chapter. These are denoted as **@:** so don't miss them if coming back. There will be no commands that need to be repeated across chapters

## 1. Live ISO

1. Download the ISO from website. [For Arch ISO](https://archlinux.org/download/)
    - For any other linux distros, install the ISO from their websites.
    - For most users, download using the WorldWide mirror (Accessible through the links before the country specific ones at the bottom)
    - If you prefer to torrent them, you can do that as well. I recommend using aria2c to download torrents.

2. Set up the disk for writing the ISO to.
    - This should be (preferably) a USB flash drive, or any other external (or internal) disk.
    - It is preferred to be in a partition separate from the one you are trying to install linux to.

3. Install the ISO to the disk.
    - Using Ventoy (recommended):
      1. Install the [Ventoy Installer](https://www.ventoy.net/en/download.html)
      2. Unzip and run the installer.
      3. Write Ventoy into the disk that you want to have the Live ISO on. (You should see a drive size when doing this. Make sure that it is the correct size you want it.)
      4. Move the ISO you downloaded into the newly created VENTOY drive.
    - Using Rufus:
      1. Install the [Rufus Installer](https://rufus.ie/en/#download)
      2. Run the installer.
      3. Select the drive and the ISO
        - **WARNING!** Rufus will usually remove the entire drive to replace with this ISO. This method is only prefered if you want to replace the entire disk.
      4. Click start.
      5. Select the DD Image mode, then click ok.

## 1.5. Setting up Windows for dual booting

Skip this section if you are not dual booting windows or not planning on keeping windows.

1. Remove Bitlocker
    - You can either stop bitlocker, or completely remove its encryption. In the latter case, it can increase read-write speeds on windows.
    - Removing bitlocker doesn't affect security as much as they want you to think. Unless you are being careless, it shouldn't matter much.

2. Create a partition
    - **WARNING!** Changing the partition size later can be difficult and time consuming. It is preferable for you to give a decent size to the linux partition to begin with.
    - You can either make a partition for linux or use another drive entirely. Either method works perfectly fine.
    1. Use the Windows Partition Manager or a 3rd party tool (Minitool Partition Wizard, or similar)
    2. Shrink the Windows partition. (Change the Microsoft Basic Data partition. Not the EFI system or Recovery.)
    - You may find that the built in partition tool cannot partition as much as the filespace that is empty. Use a disk de-fragmenter, optimiser and Minitool Partition Wizard instead.
    - **NOTE:** This partition doesn't need to be formatted, as we will be doing this later.

## 2. BIOS/UEFI Setup

### Launching into BIOS/UEFI

You have multiple ways of launching into UEFI. You should try the DEL/F2 mashing technique first. If that doesn't work, try the following in Command Prompt from windows.
```bat
shutdown /r /o /f /t 0
```
You should enter a blue Recovery menu. Here, follow the following menus.

> Troubleshoot >> Advanced Options >> UEFI Firmware Settings

### Settings to change

1. Boot/Fast boot > Disabled (Off)
2. Security/Secure Boot > Disabled (Off)

Optionally, you can enable ReBAR now, if you are on a desktop.

## 3. Boot into Live ISO

You can either change the boot priority under *Boot*, or simply launch into the ISO through *Save And Exit*.

## 4. Installing on Live ISO

This part is different on other linux distros. Only follow these steps (You really can't follow these) if you are on Arch.

**NOTE:** IF PAUSING THE INSTALL AT ANY POINT, SHUTDOWN THE COMPUTER USING THE STEPS UNDER REBOOTING RIGHT BEFORE CHAPTER 5.

### Partition Setup

We need to find the disk name that you are writing to. This will be something like **nvme0n1** Or similar. Make sure to keep track of the numbers, since these can change between sessions, but are important when doing the following steps. 
```sh
fdisk -l
```
Now we partition the disk with the \<disk name\> from above.
```sh
cfdisk /dev/<disk name>
```

Here is an example. This will be the first and only example throughout this.
```sh
cfdisk /dev/nvme0n1
```
Make sure the number is the same as the one from *fdisk -l*.

---

Running cfdisk should have opened a TUI menu. Here, we need to create a minimum of 2 partitions (EFI System and Linux filesystem).

For this guide, we will make 3.

1. EFI System ~ 1G
2. Linux swap ~ 2-32G
3. Linux filesystem ~ However much or little you want. At least 15G but with modern requirements, >50G is preferable. 

If you do not wish to have a Linux swap partition, follow the steps without running the commands for the swap partition. It will not be a problem.

---

To create these partitions, we follow the following steps.

1. Hover the cursor over the *Free space*
2. Select [ New ]
3. Enter the Partition size (EG: 8G)
4. Select [ Type ]
5. Select the correct type for the partition
6. Repeat from step 1. for each partition
7. Confirm that you have created the correct partitions, and that you haven't accidently deleted anything important.
8. Select [ Write ] and enter *yes*
9. Slect [ Quit ]

This should have created the partitions.

Check the partition numbers using 
```sh
fdisk -l
```
**NOTE:** The numbers after the p are important

---

Example partition names:

- EFI System       : /dev/nvme0n1p1
- Linux swap       : /dev/nvme0n1p2
- Linux filesystem : /dev/nvme0n1p3

If you do not have windows partitions, these numbers should be basically correct. Keep in mind the numbers. Make sure to check the numbers for the following commands as they can possibly remove your windows or other linux installs.

---

Now we format the partitions.

Run the following

**WARNING! MAKE SURE THE PARTITION NUMBERS ARE CORRECT! THIS WILL REMOVE THE DATA OF ANY PARTITIONS**
```sh
mkfs.fat -F 32 /dev/<EFI partition>
mkswap /dev/<Swap partition>
mkfs.ext4 /dev/<Filesystem partition>
```

**@:** Now that we have our partitions formatted, we mount the partitions. The order or partition numbers are different this time. Be careful. This step should be repeated if you leave before finishing chapter 4. **NOTE:** The partition numbers may change between sessions. Make sure to use fdisk -l to re-check the numbers if you leave.
```sh
mount /dev/<Filesystem partition> /mnt
mount --mkdir /dev/<EFI partition> /mnt/boot
swapon /dev/<Swap partition>
```

---

### Networking

**@:** This is the method for connecting the Live ISO to the internet for installation. Follow this method for the installer for chapter 4.

**NOTE:** You can skip this section if you have ethernet, as it should work out of the box. This method is only for those using WiFi.

Run
```sh
iwctl
```
This should get your terminal into the [iwd]# mode.

Now we run 
```iwd
device list
```
for the device name for the WiFi card. This is usually something like *wlan0*.

```iwd
station <card name> scan
station <card name> get-networks
```
The second command will display the discoverable networks. If your WiFi network doesn't show up, try the usual WiFi fixing steps or re-run the first two commands.

You can always use CTRL-C to stop a command from executing if you cannot enter the next command.

We now connect to the WiFi

```iwd
station <card name> connect <SSID>
```

We exit iwctl using
```iwd
exit
```

---

### Pacstrap

Here, we get to choose some things. We choose the kernel and CPU microcode. These options will be layed out and also given an example for the base install.

As you may (or may not) know, the kernel is the layer of connection between you and the hardware. Choosing which kernel to use is important. It should be noted that multiple kernels can be installed at once, and switched between in GRUB. If you wish to install multiple, just add all the \<kernel\> and \<headers\> in the list (with spaces in between).

These kernels can also be replaced, removed, or added at any time. You do not need to choose now, and go with stable if you would like. 

- Stable / Mainline
    - It is the newest and most bleeding edge kernel.
    - \<kernel\> = linux, \<headers\> = linux-headers
- Long-Term Support
    - This is the best for most people. If ever in doubt, go with this one.
    - This kernel is for people who have been having issues with the computers, or for those who are worried about stability on the mainline kernel.
    - If you don't care about the best of the best and want as little problems as possible, this one is the go-to.
    - \<kernel\> = linux-lts, \<headers\> = linux-lts-headers
- Performance / Zen
    - This kernel is mostly for those who care about latency. Probably just use the CachyOS kernel instead. (CachyOS kernel is to be added later in Chapter 6. Don't bother with thinking about it now.)
    - \<kernel\> = linux-zen, \<headers\> = linux-zen-headers

Now for the CPU microcode. This is basically additional code for CPU manufacturers to fix their crappy CPUs after they have sold them.

- If using an Intel CPU, \<microcode\> = intel-ucode
- If using an AMD CPU, \<microcode\> = amd-ucode


If you've chosen the options, we run the following command: (The >> is to be entered as well. Its not a note I wrote.)
```sh
pacstrap -K /mnt base <kernel> <headers> <microcode> linux-firmware man-db
genfstab -U /mnt >> /mnt/etc/fstab
```

---

Example pacstrap commands:
```sh
pacstrap -K /mnt base linux linux-headers intel-ucode linux-firmware man-db
pacstrap -K /mnt base linux-lts linux-lts-headers amd-ucode linux-firmware man-db
pacstrap -K /mnt base linux linux-lts linux-headers linux-lts-headers amd-ucode linux-firmware man-db
```

---

**@:** Now we chroot into our actual install.
```sh
arch-chroot /mnt
```

Doing this should move you from

> root@archiso ~ #

to

> [root@archiso /]#

Or something similar.

---

### Base Time

We need to set a base time for the system to work properly. For this, we will just set the time to be for the UK. Any changes to time zone can be done later using the GUI (System settings)

```sh
ln -sf /usr/share/zoneinfo/Europe/London /etc/localtime
hwclock --systohc
```

---

### Other installs

**NOTE:** You can optionally run all the installs in one pacman command. (Add the options after the -S in a list, separated by spaces)

**NOTE 2:** You can just press enter for all pacman options in this tutorial. For options that it gives, the capital letter option is the one that is chosen if no input is given.

We will be adding NetworkManager to the install.

**WARNING!** Don't forget to do this one. We will have to come back to the Live ISO and have to follow all the **@** steps back here.

```sh
pacman -S networkmanager
```

We also have a choice of text editors. Either we can use NANO or VIM/NVIM. The latter is a lot harder to use, but can be quite fun when you learn it. For this guide, we will be using NANO. For those using VIM, replace every command from now on with *vim*.

```sh
pacman -S nano
pacman -S vim nvim
```

**NOTE:** For nano, saving is done by

> CTRL-O >> Enter

and exiting through

> CTRL-X

We also need sudo. This is universally used in linux for Administrative Privilages (In linux, we refer to admin as root.)
```sh
pacman -S sudo
```

We will install Konsole (terminal emulator) and Dolphin (file manager) through pacman as well.
```sh
pacman -S konsole dolphin
```

---

### Language

We will be setting up the base language (US English). You can either choose another language now, or follow through with the same setup.

```sh
nano /etc/locale.gen
```

Here, uncomment the line (Remove the #)

> en\_US.UTF-8

Save and exit the file.

Now run
```sh
locale-gen
```
Which grabs the language we uncommented. Now we open the following file (its a different file!)
```sh
nano /etc/locale.conf
```
and add

> LANG=en\_US.UTF-8

to the beginning of the file.

---

### Computer name

The following steps is to add the system's name. This is changable later. We add the name to the following file
```sh
nano /etc/hostname
```

The system name is usually all lowercase, and cannot have spaces. Use dashes (-) instead of spaces.

Something like:

> arch-laptop

It is fully up to you to choose. We will later add the user, and the username.

We will now set the root password by running the following command, then adding the password.

**NOTE:** The password will not show up when typing. Do not be alarmed, this is normal. If you see your password showing, you are doing it wrong.
```sh
passwd
```

**WARNING!** Please don't forget this password. Like please don't...

---

### GRUB

GRUB is the main linux boot loader that most distros use. Windows dual booting can also use this, so don't worry.
```sh
pacman -S grub os-prober
```
Now we edit the following
```sh
nano /etc/default/grub
```
by uncommenting

> GRUB\_DISABLE\_OS\_PROBER=false

Make sure to keep the value as **false**

Then we set the boot loader up.
```sh
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Arch --removable
```

The following command should be run now, and (if dual booting windows) can be run later on again in chapter 6 to add windows to GRUB. (I will remind you of this)
```sh
grub-mkconfig -o /boot/grub/grub.cfg
```

---

### Rebooting

To safely reboot, we need a few more steps than just hitting the power button. 

For those here finishing chapter 4, make sure to enter BIOS/UEFI by the mashing technique. Make sure to remove the Live ISO device during the reboot.

If you are here to pause and shutdown, replace *reboot* with *poweroff*

```sh
exit
umount -R /mnt
reboot
```

## 5. Actual Install

Now that we are back into BIOS/UEFI, we set GRUB (Usually just the name of the SSD and "EFI SYSTEM"). For those without another linux system or windows, it should be the only thing there.

If we save and exit, we should be greeted with the grub menu, which can be paused by pressing a button. It should say Arch at the top, with a "Advanced Arch" option, and a UEFI option. Enter the first Arch option normally, and you should see your chosen kernel at the top before everything comes in.

The login options should be

Username:

> root

Password from before (*I did say it's important!*)

> \<password\>

---

### User

We need to add a user profile. The naming method is the same syntactically (all lower case, - instead of space).

Most people don't add the "arch" this time, however.

```sh
useradd -m -G wheel <username>
passwd <username>
```

The password command is the same as before, it will not show up when typing.

This user is under the "wheel" group, which is the basic user group. We need to give it sudo permissions for editing files outside of the home directory (root directory).

Open the file by running
```sh
EDITOR=nano visudo
```
and uncomment either

> %wheel ALL=(ALL:ALL) ALL

or

> %wheel ALL=(ALL:ALL) NOPASSWD: ALL

depending on whether you want to be asked for your password (user password) whenever you want to run with sudo.

If you ever want to edit this afterwards, comment the current one, and uncomment the other.

For personal computers, I recommend uncommenting the second one, as it becomes quite annoying to type the password each time.

---

### Networking

We're back to networking. We activate Network Manager that we installed earlier (I said its important! No.2).
```sh
systemctl enable --now NetworkManager
```

We can check if the device is working correctly with
```sh
nmcli radio wifi
```

We can find the device name (different from before!) with
```sh
nmcli device wifi list
```

Take the device name with type *wifi*. It should be *disconnected*. It is usually something like *wlp1s0* or something similar. The L and 1 look very similar, so be mindful.

If you need to take a look at the accessible wifi networks, run
```sh
nmcli device wifi list
```

Then we connect using
```sh
nmcli device wifi connect <SSID> password <password> ifname <device name>
```

### Desktop Environment

So we're finally here. Adding a GUI! For this guide, we will be installing and configuring KDE plasma wayland. For other environments, find out about it and install it yourselves. For programmers, take a look at hyprland. For most people, KDE is the best choice, but feel free to try out GNOME or cinnamon as well.

```sh
pacman -S xorg plasma sddm
```

---

For those with an nvidia GPU, firstly, my condolences, but importantly, we need to choose which nvidia driver to install based on the kernel.

> Normal kernel = nvidia-open

> LTS kernel = nvidia-open-lts

> ZEN kernel = nvidia-open-zen

**NOTE:** CachyOS also has its own nvidia driver.

Alternatively, we can install the DKMS version, which is compatible for all kernels. (This will take a lot of time for each kernel/driver based install/update, unlike getting the specific driver)

> Any kernel = nvidia-open-dkms

This is done by running
```sh
pacman -S <nvidia driver> nvidia-utils nvidia-settings
```

Example:
```sh
pacman -S nvidia-open-dkms nvidia-utils nvidia-settings
```

---

Now that we have our desktop environment, we can enable sddm then reboot.
```sh
systemctl enable sddm
reboot
```

## 6. Other things!

Good job! In the case you got here correctly, you should see a login screen. **DON'T LOG IN YET!** At the top left, change "Plasma Bigscreen" to "Plasma (Wayland)".

Now that we've done the hard stuff, we should be left with a plain old KDE plasma... Which is why you get to explore KDE plasma settings for all the wonders.

---

### Installing apps

The way to install apps are 3 ways.
1. pacman
  - Pacman is the base package installer. For most installs, search it up to see if there is a pacman install. (Something like: "\<App name\> pacman install")
2. yay (AUR)
  - Yay is the Arch User Repository installer. This is for packages on git, and other things, that clones the repo and installs it with pacman in the back. 
  - The way to install yay is given later.
3. Flatpak (Discover)
  - A GUI method of installing apps. The easiest, but usually with some caviats that are really annoying. If you ever run into problems with an app, usually blame Flakpak and its amazing user security.

For most apps, install using Flatpak if it doens't need any high level access to files, etc. If you want max performance, use pacman/yay.

**Important!** Install flatseal (on flatpak/discover) as it allows you to get more control over the app's permissions.

---

### yay

Installing yay is quite simple. (You might want to install firefox first, to be able to copy and paste these commands.

```sh
sudo pacman -S --needed base-devel git

cd ~/Downloads
git clone https://aur.archlinux.org/yay-bin.git
cd yay-bin
makepkg -si
```

Verify it installed properly with
```sh
yay --version
```

If it installed properly, run
```sh
cd ~/Downloads
rm -rf yay-bin
```

**NOTE:** When installing with yay, it'll ask for multiple options and things. Just press enter through them.

---

### 32-bit apps

Some pacman apps are 32-bit based. We can (and should) enable these by opening
```sh
sudo nano /etc/pacman.conf
```
then uncommenting

> [multilib]

> Include = /etc/pacman.d/mirrorlist

---

### Steam

For gamers that use Steam (basically everyone, no?), you can install with pacman (don't use flatpak as it can cause problems).

First enable 32-bit apps as above, then run
```sh
sudo pacman -S steam
```

---

### Nvidia 32-bit libs

For those with Nvidia GPUs, we should also install
```sh
sudo pacman -S lib32-nvidia-utils
```

---

### Nvidia settings

The newer nvidia drivers have some issues with options that need to be added. (These may be depricated now. Make sure to add at your own risk)
```sh
sudo nano /etc/modprobe.d/nvidia.conf
```
> options nvidia NVreg\_EnableGpuFirmware=0

> options nvidia NVreg\_PreserveVideoMemoryAllocations=1

It might help, it might not.

---

### KDE Themes

Basically everything on KDE, including some apps (like Konsole) have externally downloadable themes. These are all on the [KDE Theme Store](https://store.kde.org/browse) or downloadable from the *Get New* option in most tabs that allow downloadable themes. 

The main theme can be accessible from

> System Settings >> Colors & Themes >> Global Theme

Here, the three dots allow you to access the KDE Theme store as well. 

The most popular theme is the Sweet theme. You should explore other options as well, if you'd like.

---

### Animated cursors

Those who really like to increase their computers process count, you can make windows .ani cursors into linux .xcur cursors with a simple app.

```sh
yay -S ani2xcursor
```

Then for each cursor you want to change into xcursors,

```sh
ani2xcursor <cursors directory> -s <sizes> --install -o <output directory>
```

For example:
```sh
ani2xcursor . -s 16,24,32,42,48,64,96 --install -o ./output_theme
```
This should be ran where the .ani cursors are.

---

### Terminal

Want cool looking terminal?

 Install a [nerd font](https://www.nerdfonts.com/font-downloads).

We can also add oh-my-posh with:

Install with
```sh
yay -S oh-my-posh-bin
```
and find a OMP theme you would like, (Some themes from the [website](https://ohmyposh.dev/docs/themes))
```sh
nano ~/.bashrc
```
> eval "$(oh-my-posh init bash --config $HOME/.cache/oh-my-posh/themes/\<theme\>)"

Move your installed theme to the location as above. Sometimes, you might need to create the directory first.
```sh
mkdir -p ~/.cache/oh-my-posh/themes
cd ~/Downloads
mv <theme> ~/.cache/oh-my-posh/themes
```

You can also adjust the themes through the konsole settings, and setting the font there. Make a profile, and edit the theme. You can get additional themes, blur the background and increase transparency and such.

Want a floating terminal? Set a default location with

> ALT-F3 >> More Actions >> Configure Special Application Settings...

Set a Size and Position, set no Titlebar and Frame (You can access it with ALT-F3 at any point), then you have a floating terminal!

You can remove the konsole bar at the top, by

> Right click (anywhere) >> Menu >> Settings >> Toolbars Shown >> Disable both

We can also add rounded corners and a slight boarder effect with
```sh
yay -S kwin-effect-rounded-corners-git
```
and

> System Settings >> Window Management >> Desktop Effects >> Rounded Corners

and untick *Disable Roundness on Tile*

Happy configuring!

---

### BTOP

Want a better System Monitor? Tired of it showing random values? Install BTOP

```sh
sudo pacman -S btop
```

There are other alternatives to btop, like go-top and more. Try them all out, see which one you like most!

---

### Systemctl

Systemctl is the command used for managing applications for startup and others.

```sh
sudo systemctl enable <service>
```

Allows the service to be ran on startup.

```sh
sudo systemctl start <service>
```
just runs the service for this boot.

Combining both is
```sh
sudo systemctl enable --now <service>
```
allows you to have the service startup on boot and have it run for this boot as well.

---

### Ranger

Annoyed at moving through folders and directories using cd? You can use ranger to move using vim commands!

Most of the time, cd mashing is the only real way you need and you can also open a terminal through dolphin, but ranger can help a lot.

```sh
sudo pacman -S ranger
```

When you want to run it,
```sh
. ranger
```

This allows you to cd on exit with :q.

---

### CN/JP/KR Characters

These are not installed by default. You should install them with
```sh
sudo pacman -S noto-fonts-cjk
```
otherwise, they will appear as squares.

---

### JP/KR Keyboards

I'm only familiar with these. For CN users, this process is probably similar.

JP keyboard is *mozc*

KR keyboard is *hangul*

Run (as needed)
```sh
sudo pacman -S fcitx5-im fcitx5-mozc fcitx5-hangul fcitx5-configtool
```

then enable in
```sh
sudo nano /etc/environment
```
and add

> GTK\_IM\_MODULE=fcitx

> QT\_IM\_MODULE=fcitx

> XMODIFIERS=@im=fcitx

and apply the fcitx5 in

> Keyboard >> Virtual Keyboard

restart, then

> Input Method >> Add Input Method

and add mozc/hangul

---

### Tailscale

A very useful tool when wanting to connect devices on a pseudo-LAN for a lot of different projects and other things.

```sh
sudo pacman -S tailscale
sudo systemctl enable --now tailscaled
sudo tailscale up
```

Allows you to connect to a tailnet.

I suggest you to find out more about tailscale if you are interested.
