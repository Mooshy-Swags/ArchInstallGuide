# Arch Install

## Partition

```sh
fdisk -l
cfdisk /dev/<disk>
mkfs.ext4 /dev/<root>
mkswap /dev/<swap>
mkfs.fat -F 32 /dev/<boot>
```

## Mount

```sh
mount /dev/<root> /mnt
mount --mkdir /dev/<boot> /mnt/boot
swapon /dev/<swap>
```

## Network

```sh
iwctl
```

```iwd
device list
station wlan0 scan
station wlan0 get-networks
station wlan0 connect <SSID>
```

## Pacstrap

```
- Mainstream        >> linux linux-headers
- Long Term Support >> linux-lts linux-lts-headers
- Zen               >> linux-zen linux-zen-headers
```

```
- Intel CPU >> intel-ucode
- AMD CPU   >> amd-ucode
```

```sh
pacstrap -K /mnt base <kernel> <headers> linux-firmware man-db vim nvim networkmanager sudo konsole
genfstab -U /mnt >> /mnt/etc/fstab
```

## Chroot

```sh
arch-chroot /mnt
```

## Time

```sh
ln -sf /usr/share/zoneinfo/Europe/London /etc/localtime
hwclock --systohc
```

## Locale

```sh
vim /etc/locale.gen
```

> en\_US.UTF-8

```sh
locale-gen

vim /etc/locale.conf
```

> LANG=en\_US.UTF-8

## Base name

```sh
vim /etc/hostname
```

> <hostname>

```sh
passwd
```

## GRUB

```sh
pacman -S grub os-prober
vim /etc/default/grub
```
> GRUB\_DISABLE\_OS\_PROBER=false

```sh
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Arch --removable
grub-mkconfig -o /boot/grub/grub.cfg
```

## Reboot

```sh
exit
umount -R /mnt
reboot
```

## Login

> root

> \<password\>

## User Setup

```sh
useradd -m -G wheel <username>
passwd <username>
```

> \<password\>

```sh
EDITOR=vim visudo
```

> %wheel ALL=(ALL:ALL) NOPASSWD ALL

## Network

```sh
system enable --now NetworkManager
nmcli device
nmcli device wifi list
nmcli device wifi connect <SSID> password <WiFi password> ifname <device name>
```

## Desktop

```sh
pacman -S xorg plasma sddm
```

## Nvidia GPU

```
- Mainstream        >> nvidia-open
- Long Term Support >> nvidia-open-lts
- Zen               >> nvidia-open-zen
- DKMS              >> nvidia-open-dkms
```

```sh
pacman -S <nvidia driver> nvidia-utils nvidia-settings
```

# Other things

## JP Font

```sh
sudo pacman -S noto-fonts-cjk fcitx5-im fcitx5-mozc fcitx5-configtool
sudo vim /etc/environment
```

```
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
```

> System settings >> Keyboard >> Virtual Keyboard >> fcitx 5

```sh
reboot
```

> System settings >> Input Methods >> Add Input Method >> mozc

## 32bit

```sh
sudo vim /etc/pacman.conf
```

> enable multilib

```sh
sudo pacman -Syu
```

## Steam

```sh
sudo pacman -S steam
```

## 32bit Nvidia

```sh
sudo pacman -S lib32-nvidia-utils
```

## Yay

```sh
sudo pacman -S --needed base-devel git
cd ~/Downloads
git clone https://aur.archlinux.org/yay-bin.git
cd yay-bin
makepkg -si
yay --version
```

## OMP

```sh
yay -S oh-my-posh-bin
echo 'eval "$(oh-my-posh init bash --config $HOME/.cache/oh-my-posh/themes/velvet-modified.omp.json)"' >> ~/.bashrc
mkdir -p ~/.cache/oh-my-posh/themes
mv ~/Downloads/velvet-modified.omp.json ~/.cache/oh-my-posh/themes
```

## Konsole Settings

```sh
yay -S kwin-effect-rounded-corners-git
```

> System settings >> Window Management >> Desktop Effects >> Rounded Corners >> Untick Disable Roundedness on corners

## Cargo

```sh
echo 'export PATH="$HOME/.cargo/bin:$PATH"' >> ~/.bashrc
```

## Nvidia settings

```sh
sudo vim /etc/modprobe.d/nvidia.conf
```

```conf
options nvidia NVreg_EnableGpuFirmware=0
options nvidia NVreg_PreserveVideoMemoryAllocations=1
```

## Cursors

```sh
yay -S ani2xcursor
```

```sh
ani2xcursor <dir> -s <sizes> --install -o <output>
```

```sh
ani2xcursor . -s 16,24,32,42,48,64,96 --install -o ./output_theme
```

## JP Audio

```sh
mkdir -p clean_output && for f in *.mkv; do mkvmerge -o "clean_output/$f" --audio-tracks jpn "$f"; done
```

```sh
touch videochanger
echo 'mkdir -p clean_output && for f in *.mkv; do mkvmerge -o "temp.mkv" --audio-tracks jpn --no-attachments "$f"; ffmpeg -i "temp.mkv" -map 0:v:0 -map 0:a:0 -map 0:s? -vf "scale=-2:720,format=yuv420p10le" -c:v libx265 -crf 26 -preset fast -x265-params aq-mode=3 -c:a copy -c:s copy "clean_output/$f"; rm "temp.mkv"; done' >> videochanger
chmod +x videochanger
sudo mv videochanger /usr/local/bin
```

## SSH

```sh
sudo pacman -S openssh
sudo systemctl enable --now sshd

ssh-keygen -t ed25519 -C <label>
```
> Set file name in ~/.ssh/
```sh
ssh-copy-id -i <public key> <user>@<server ip>
```

> In ~/.ssh/config add:
```
Host <Hostname>
  HostName <Hostname>
  User <user>
  IdentityFile ~/.ssh/<private key>
```

## Sending files over ssh

```sh
sudo pacman -S rsync openssh
sudo systemctl enable --now sshd
```

```sh
rsync -avzP <dir> <usr>@<ip>:<dir>
```

```sh
rsync -avzP ./* mooshy@100.94.245.111:/home/mooshy/Downloads/Anime
```

## Renaming files

```sh
for f in *.mkv; do
  [ -f "$f ] || continue
  ep="${f: inx:len}"
  mv -n -- "$f" "title - SXXE${ep}.mkv"
done
```

## Youtube download

```sh
sudo pacman -S yt-dlp
```

```sh
yt-dlp -f bestaudio -x --audio-format mp3 --embed-thumbnail --add-metadata --cookies-from-browser firefox “<link>”
```

## CachyOS

```sh
curl -O https://mirror.cachyos.org/cachyos-repo.tar.xz
tar xvf cachyos-repo.tar.xz
cd cachyos-repo
sudo ./cachyos-repo.sh
sudo pacman -Sy linux-cachyos-lts linux-cachyos-lts-headers
sudo pacman-key --recv-keys F3B607488DB35A47 --keyserver keyserver.ubuntu.com
sudo pacman-key --lsign-key F3B607488DB35A47
sudo pacman -Syyu
sudo pacman -Scc
sudo pacman -Syy
pacman -Qqn | sudo pacman -S -
```

Nvidia GPU:
```sh
sudo pacman -S linux-cachyos-lts-nvidia-open nvidia-utils lib32-nvidia-utils
```

## Btop preset

```
cpu:0:default,mem:0:default,gpu0:0:default
```

## Turn off screen keybind

> System settings >> Keyboard >> Shortcuts >> Power Management >> Turn Off Screen

## User BIN files

```sh
vim ~/.bashrc
```

```
export PATH="$PATH:/home/mooshy/.local/bin:$HOME/bin"
```

## Tailscale

```sh
sudo pacman -S tailscale
sudo systemctl enable --now tailscaled
sudo tailscale set --operator=$USER
```

## Git

setup
```sh
git config --global user.name "<username>"
git config --global user.email "<email>"

git config --global credential.helper store

ssh-keygen -t ed25519 -C "<email>"
```
> /home/\<user\>/.ssh/github

> copy .pub to github >> settings >> SSH and GPG

workflow
```sh
git clone git@github:<username>/<repo>

git checkout -b <branch>
# >> do stuff
git add <files>
git commit -m <message>
git push -u origin <branch>

gh pr create --fill
gh pr merge --merge

git checkout main
git pull
git branch -d <branch>
```


## Osu:

Size: 6,17; 44,41
