# Arch Install Guide

This guide is simple, but goes through all of the important ideas for installing Arch.

## Live ISO

To install any linux or windows version, you must create a live ISO containing the installer. This can be done from windows by using a tool like rufus or ventoy to write into a usb stick. Ventoy is preferred if you like to keep the rest of the drive being written to, but rufus is usally easier to use. (Use DD mode for rufus)

## Dual Booting

**please for the love of god, remove fucking bitlocker before doing anything.**

Start from windows, create an empty partition if using one drive for the dual boot. Otherwise, skip this step.

# Arch install

Open into BIOS/UEFI (UEFI is an upgraded version of BIOS that most modern systems come with) using:

```bat
shutdown /r /o /f /t 0
```

## BIOS/UEFI setup

Turn fast boot and secure boot off. (Requires UEFI admin)

Boot into Arch Install Media (usually USB flash drive)

## Boot Media

### Partition setup

Find the disk name (Something like "nvme0n1") which is the disk you would like to install to using:

```sh
fdisk -l
```

Now partition the disk using (replace <disk name> with what you got above.):

```sh
cfdisk /dev/<disk name>
```

Partition should have a minimum of 2 parts. For this guide, I will only follow with 3. 

Swap partition is not required but is useful for anything beyond 100GB of filesystem or low RAM devices. For low end usecases, its not needed.


Create a **efi\_system\_partition** with size ~1G

Create a **swap\_partition** with size ~8G - 16G

Create a **linux\_filesystem** with the rest of the space.


*Make sure the types are correct, otherwise the next step will fail.*


Now use
```sh
fdisk -l
```
again, keeping note of the partitions.

---

EG: (The <partition name> will be kept throughout the following part using \*)

efi\_system\_partition : nvme0n1p1
swap\_partition        : nvme0n1p2
linux\_filesystem      : nvme0n1p3

---

The files will give correct errors if you try to format the wrong partitions, since we made them the correct format with cfdisk. However, mounting needs to be done carefully.

Now for formatting.

```sh
mkfs.ext4 /dev/*nvme0n1p3*
mkswap /dev/*nvme0n1p2*
mkfs.fat -F 32 /dev/*nvme0n1p1*
```

Now we mount the partitions, so the system knows what partitions are together.

```sh 
mount /dev/*nvme0n1p3* /mnt
mount --mkdir /dev/*nvme0n1p1* /mnt/boot
swapon /dev/*nvme0n1p2*
```

We are done setting the partitions.

### Networking

We are now going to connect to the internet **for the installer**. (Note: we will need to re-connect to the internet for our Arch Install later)

```sh
iwctl
```

This gets you into the network connecting service.

Now we run

```iwctl
device list
```

to get the device name for the WiFi card.

---

The wifi card is usually called wlan0, so we proceed with this name.

```iwctl
station *wlan0* scan
station *wlan0* get-networks
```

This will display all the connectable/findable WiFi networks. Make sure to note down which one you will be connecting to.

```iwctl
station *wlan0* connect *ssid*
```

You can now exit iwctl by using:

```iwctl
exit
```


### Installing Kernel


Run the following (unless you are using an **INTEL CPU**, in which cause, replace with *intel-ucode* 

If you wish to install the lts (Long time support) kernel as well, add "linux-lts linux-lts-headers"

```sh
pacstrap -K /mnt base linux linux-firmware linux-headers *amd-ucode* man-db
genfstab -U /mnt >> /mnt/etc/fstab
```

These are important steps. Not including any one of these properly can break the system. If misspelt, re-run the correct command, though un-ideal.

Now we "Enter" our install with:

```sh
arch-chroot /mnt
```

### Set base Time

We can change the local time afterwards in UI, so we just set it to the UK for ease.

```sh
ln -sf /usr/share/zoneinfo/Europe/London /etc/localtime
hwclock --systohc
```

### Important Installs

We now need to add the network manager for our install. **MAKE SURE THIS IS DEFINITELY INSTALLED NOW, OR WE NEED TO RE-ENTER THE INSTALLER AGAIN WHICH IS ANNOYING.**

```sh
pacman -S NetworkManager
```

We also choose the terminal text editor now. The main choices are Nano and Vim (or Neo-Vim). Nano is more "normal" in the operations. Most things should be more obvious (Though I personally dislike it).
**IF AT ANY POINT YOU SEE vim, REPLACE WITH THE CORRECT ONE YOU INSTALLED.

```sh
pacman -S nano
pacman -S vim nvim
```

Basic Nano operations is Ctrl+O -> Enter for saving, and Ctrl+X to exit. For VIM usage, refer to the (Neo)VIM page or find it online.

Finally, we need to make sure we have admin for downloading things later.

```sh 
pacman -S sudo
```

### Language

We now setup our base language. Any other keyboard can be added later on, so we will just run with US.

```sh
vim /etc/locale.gen
```

Now uncomment the following line:

```gen
en_US.UTF-8
```

and save and exit out of the file.

Now run

```sh
locale-gen
```

which sets the language up to be added. Now we need to add it. (The file is different so be careful)

```sh
vim /etc/locale.conf
```

and add:

```conf
LANG=en_US.UTF-8
```

### Computer name

We need to set the computer name.

```sh
vim /etc/hostname
```

Now we add the name into the file, then save and exit. We can change this later if you wish, though it may be hard. Make sure you have a name you are satisfied with.

We also set a password for the root. Don't fuck this one up. Please don't. And remember it. Remember it please.

The command will prompt you to add the password after the command. **DO NOT** put it as the argument.

```sh
passwd
```

### GRUB

GRUB is the main linux boot loader. We need this to load into arch, and also to set up the Windows dual boot later. The windows can be dealt with later, so we will just do arch to begin with.

```sh
pacman -S grub os-prober
vim /etc/default/grub
```

Uncomment the following (it is usually near the bottom)

```
GRUB_DISABLE_OS_PROBER=false
```

Make sure it is **FALSE**

Now we just run the following two commands:

```sh
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Arch --removable
grub-mkconfig -o /boot/grub/grub.cfg
```

### Rebooting

**DON'T JUST REBOOT YET**

Run the following commands to reboot instead.

```sh
exit
umount -R /mnt
reboot
```

This safely exits the mounted install, and unmounts for the installer safely.

## Actual Install

We're finally here, on our actual arch install. If you followed up to here, we should be quite smooth sailing from here.

Login using the username: root

and the password from before. (I said it's important.)

### User

We also add a user profile. This password is the one you will be using to normally log into the user. Make sur ethis one is definitely one you remember. Unfortunately, linux doesn't really allow you to use short passwords, fingerprints or face recognition.

```sh
useradd -m -G wheel <username>
passwd <username>
```

Now we run:

```sh
EDITOR=vim visudo
```

and uncomment the line:

```
%wheel ALL=(ALL) ALL
```

This allows us to actually use admin and sudo in our user space.

### Network

We're back to networking. 

```sh
systemctl enable NetworkManager
systemctl start NetworkManager
```

This starts our Network Manager. **RUNNING BOTH COMMANDS IS ESSENTIAL** Now we run

```sh
nmcli radio wifi
```

to get our wifi device name. It is possibly different to what we got before, so be careful. We will call it <device name>

```sh
nmcli device
nmcli device wifi list
```

Make sure to note the name of the network here. You can exit using CTRL+C

```sh
nmcli device wifi connect <SSID> password <network password> ifname <device name>
```

Wow, we had 3 things to change there! How fun.

### Desktop

Wow, after 330 lines of this MarkDown file, we're finally adding the desktop environment. This guide will run through KDE Plasma, but feel free to install GNOME or Cinnamon if you prefer. I personally will always encourage using KDE.

```sh
pacman -S xorg plasma sddm
```

Now if you have an NVIDIA GPU, run the following:

(If you don't have the lts version, you can use nvidia-open instead of dkms in the case of the nvidia drivers being weird.)

```sh
pacman -S nvidia-open-dkms nvidia-utils nvidia-settings
```

We are finally at the point of enabling our desktop environment. **check that all previous steps have been completely executed properly (especially for the Networking inside the install) otherwise, we may have issues**

Good? Okay, we start our desktop environment:

```sh
systemctl enable sddm
reboot
```

## Other things

Good job! You may have gotten to this point successfully. If you have, we're now done with the hard stuff, we just have customisations and adding other apps!

The main method of installing apps is using the "discover" app, which runs flatpak, however, some things may need or benefit from using pacman or yay.

The apps you should download is:

 - Firefox (Browser)
 - Dolphin (File Explorer)
 - Konsole (Terminal Emulator)
 - Flatseal (Grants permissions for flatpak installed apps)

And some that you may want:

 - Vesktop (Better, moddable discord)
 - Rnote (Note taking)
 - VLC (Video playing GOAT)
 - Honkers Star railway (ykyk)

### Steam

Our favourite! But installing from flatpak can cause issues with not properly using the GPU. So we install using our friend pacman.

```sh
sudo vim /etc/pacman.conf
```

Uncomment the following lines:

```conf
[multilib]
Include = /etc/pacman.d/mirrorlist
```

Now we can just install from pacman properly.

```sh
sudo pacman -S steam
```

It is also important that you use Proton 9 as the compatability layer. This can be done through right-clicking the game, properties, compatability, then checking the check box and selecting the version.

### YAY

The Arch User Repository is basically a community version of packages that are installed using yay instead of pacman. The usage is basically the same, but requires you to install it separately.

```sh
sudo pacman -S --needed base-devel git

cd ~/Downloads
git clone https://aur.archlinux.org/yay-bin.git
cd yay-bin
makepkg -si
```

Check if YAY installed correctly using:
```sh
yay --version
```

You can delete the git folder with:
```sh
cd ~/Downloads
rm -rf yay-bin
```

