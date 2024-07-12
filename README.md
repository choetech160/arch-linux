Step-by-Step Guide to Install Arch Linux with Hyprland 



#### 1. Prepare the Installation Media
1. **Download the Arch Linux ISO**:
   - Visit [Arch Linux Download](https://archlinux.org/download/) and download the latest ISO.

2. **Create a Bootable USB Drive**:
   
   - Use `dd`:
   
   ```bash
   sudo dd if=/path/to/archlinux.iso of=/dev/sdX bs=4M status=progress oflag=sync
   ```
   or `popsicle usb flasher`
   

#### 2. Boot from the USB Drive
1. **Insert the USB Drive** and restart your computer.
2. **Enter BIOS/UEFI Settings** (usually by pressing F2, F12, Del, or Esc during startup).
3. **Set Boot Priority** to the USB drive and save changes.
4. **Select "Arch Linux install medium"** from the boot menu and press Enter.

#### 3. Set Up the Installation Environment
1. **Verify Boot Mode**:
   
   ```bash
   ls /sys/firmware/efi/efivars
   ```
   If the directory exists, you're in UEFI mode.
   
2. **Connect to the Internet**:
   - For Ethernet: It should connect automatically.
   - For Wi-Fi:
     ```bash
     iwctl
     device list
     station wlan0 scan
     station wlan0 get-networks
     station wlan0 connect "Your Network Name"
     exit
     ```

3. **Update System Clock**:
   ```bash
   timedatectl set-ntp true
   ```

#### 4. Partition the Disk
1. **Identify Your Disk**:
   
   ```bash
   lsblk
   ```
   Note the name of your disk (e.g., `/dev/nvme0n1`).
   
2. **Start Partitioning**:
   ```bash
   gdisk /dev/nvme0n1
   ```

3. **Create a New Partition Table**:
   - Type `o` and press Enter.
   - Type `Y` to confirm.

4. **Create EFI System Partition**:
   - Type `n` and press Enter.
   - Press Enter for default partition number.
   - Press Enter for default first sector.
   - Type `+512M` for the last sector.
   - Type `ef00` for the partition type.

5. **Create Boot Partition**:
   - Type `n` and press Enter.
   - Press Enter for default partition number.
   - Press Enter for default first sector.
   - Type `+1G` for the last sector.
   - Type `8300` for the partition type.

6. **Create Root Partition**:
   - Type `n` and press Enter.
   - Press Enter for default partition number.
   - Press Enter for default first sector.
   - Press Enter for default last sector (use remaining space).
   - Type `8300` for the partition type.

7. **Write the Partition Table**:
   - Type `w` and press Enter.
   - Type `Y` to confirm.

#### 5. Set Up Disk Encryption
1. **Encrypt the Root Partition**:
   ```bash
   cryptsetup luksFormat /dev/nvme0n1p3
   ```
   Type `YES` (in all caps) and enter a strong passphrase.

2. **Open the Encrypted Partition**:
   ```bash
   cryptsetup open /dev/nvme0n1p3 cryptroot
   ```

3. **Format Partitions**:
   ```bash
   mkfs.fat -F32 /dev/nvme0n1p1
   mkfs.ext4 /dev/nvme0n1p2
   mkfs.ext4 /dev/mapper/cryptroot
   ```

4. **Mount Partitions**:
   
   ```bash
   mount /dev/mapper/cryptroot /mnt
   mkdir /mnt/boot
   mount /dev/nvme0n1p2 /mnt/boot
   mkdir /mnt/boot/efi
   mount /dev/nvme0n1p1 /mnt/boot/efi
   ```

#### 6. Install the Base System
1. **Install Essential Packages**:
   
   ```bash
   pacstrap /mnt base base-devel linux linux-firmware lvm2 vim
   ```
   
2. **Generate fstab**:
   ```bash
   genfstab -U /mnt >> /mnt/etc/fstab
   ```

3. **Chroot into the New System**:
   ```bash
   arch-chroot /mnt
   ```

#### 7. Configure the System
1. **Set Time Zone**:
   ```bash
   ln -sf /usr/share/zoneinfo/Your_Region/Your_City /etc/localtime
   hwclock --systohc
   ```

2. **Localization**:
   ```bash
   vim /etc/locale.gen
   ```
   Uncomment `en_US.UTF-8 UTF-8` and any other needed locales, then generate locales:
   ```bash
   locale-gen
   echo "LANG=en_US.UTF-8" > /etc/locale.conf
   ```

3. **Set Hostname**:
   ```bash
   echo "myhostname" > /etc/hostname
   ```

4. **Configure Hosts File**:
   ```bash
   vim /etc/hosts
   ```
   Add the following lines:
   ```
   127.0.0.1   localhost
   ::1         localhost
   127.0.1.1   myhostname.localdomain myhostname
   ```

5. **Set Root Password**:
   ```bash
   passwd
   ```

6. **Create a New User**:
   ```bash
   useradd -m -G wheel -s /bin/bash myusername
   passwd myusername
   ```

7. **Configure Sudo**:
   ```bash
   EDITOR=vim visudo
   ```
   Uncomment the line: `%wheel ALL=(ALL) ALL`

#### 8. Install and Configure Bootloader
1. **Install GRUB and efibootmgr**:
   ```bash
   pacman -S grub efibootmgr
   ```

2. **Install GRUB**:
   ```bash
   grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
   ```

3. **Configure GRUB for Encryption**:
   ```bash
   vim /etc/default/grub
   ```
   Add `cryptdevice=UUID=device-UUID:cryptroot root=/dev/mapper/cryptroot` to the `GRUB_CMDLINE_LINUX` line (replace `device-UUID` with the UUID of `/dev/nvme0n1p3`, which you can find by running `blkid`).

   ```bash
   blkid | grep /dev/nvme0n1p3 | awk -F'"' '{print $2}' # find the UUID
   sed -i "s|GRUB_CMDLINE_LINUX=\"|GRUB_CMDLINE_LINUX=\"cryptdevice=UUID=$(blkid | grep /dev/nvme0n1p3 | awk -F'"' '{print $2}'):cryptroot root=/dev/mapper/cryptroot |" /etc/default/grub # Be as leasy as I am
   ```
   
4. **Generate GRUB Config**:
   
   ```bash
   grub-mkconfig -o /boot/grub/grub.cfg
   ```

#### 9. Install Hyprland and Necessary Packages
1. **Install Hyprland and Other Packages**:
   ```bash
   sudo pacman -S hyprland kitty waybar wofi dunst thunar nitrogen pavucontrol network-manager-applet blueman polkit-gnome gnome-keyring git xdg-user-dirs
   ```

   run 

   ```bash
   xdg-user-dirs-update
   ```
   
   once logged in to create folders (Desktop, Download etc)
   
    
   
2. **Clone end_4's Dotfiles**:

   - Clone the dotfiles repository:

     

     ```bash
     git clone https://gitlab.com/stephan-raabe/dotfiles.git
     cd dotfiles
     ./install.sh
     sudo pacman -S sddm
     sudo systemctl enable sddm
     ```

   **Start Hyprland**: Start Hyprland by adding it to your `.xinitrc` or using a display manager.


#### 10. Set Up Display Manager
1. **Install SDDM**:
   ```bash
   pacman -S sddm
   ```

2. **Enable SDDM**:
   ```bash
   systemctl enable sddm
   ```

#### 11. Final Steps

1. **Exit the Chroot Environment**:
   ```bash
   exit
   ```

2. **Unmount All Partitions**:
   ```bash
   umount -R /mnt
   ```

3. **Reboot the System**:
   ```bash
   reboot
   ```

#### 12. Set Up YubiKey for Boot Integrity

Follow this [procedure](https://github.com/choetech160/arch-boot-integrity-check.git)

#### 13. Set up USBGuard

Follow this [procedure](https://github.com/choetech160/arch-usbguard)

#### 14. Set up Wazuh

Follow this [procedure](https://github.com/choetech160/arch-linux/blob/main/wazuh/README.md)

#### 14. AIDE (Advance Intrusion Detection Environment)

AIDE helps detect unauthorized changes to system files, potentially indicating a compromise.

Install AIDE: 

````bash
sudo pacman -S aide
````

Initialize database and move it:

```bash
sudo aide --init
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db
```

Create a daily check:

```bash
sudo vim /etc/systemd/system/aide-check.service
sudo vim /etc/systemd/system/aide-check.timer
```

Enable the timer:

```bash
sudo systemctl enable aide-check.timer
```

#### 15. Firewall

A firewall controls incoming and outgoing network traffic, protecting against unauthorized access and potential threats. 

Install ufw

```bash
sudo pacman -S ufw
sudo systemctl enable ufw && sudo systemctl start ufw
```

Set the default policies:

```bash
 sudo ufw default deny incoming
 sudo ufw default allow outgoing
```

Disable what you don't need:

```bash
sudo ufw block ssh
```

Enable the firewall: 

```bash
sudo ufw enable
```

#### 16. Enable kernel security features (KASLR, SMAP, SMEP)

To enable the kernel security features and harden the kernel, follow [this procedure](https://github.com/choetech160/arch-linux/tree/main/Kernel-Security-Features)

#### 17. Secure the bootloader

To encrypt the boot loader, follow [this procedure](https://github.com/choetech160/arch-linux/tree/main/bootloader)

## Connect to the internets once log in without gui

First, ensure NetworkManager is installed and running:

```bash
sudo systemctl start NetworkManager
sudo systemctl enable NetworkManager
```

Use the command-line tool `nmcli` to connect to Wi-Fi: a. List available Wi-Fi networks:

```bash
nmcli device wifi list
```

b. Connect to your Wi-Fi network:

```bash
nmcli device wifi connect "Your Network Name" password "Your Password"
```

Replace "Your Network Name" with your actual Wi-Fi network name (SSID) and "Your Password" with your Wi-Fi password.

Verify the connection:

```bash
nmcli connection show
```

## Audio

Modern Arch Linux systems typically use  PipeWire as the audio server, which includes a PulseAudio compatibility  layer. This means PipeWire can handle PulseAudio connections without  needing to install PulseAudio separately.

To set up audio on your Arch Linux system with Hyprland, you should install PipeWire and its related packages:

```bash
sudo pacman -S pipewire pipewire-alsa pipewire-pulse pipewire-jack wireplumber
```

This will install PipeWire and its compatibility  layers for ALSA, PulseAudio, and JACK, as well as the session manager  WirePlumber.

After installation, enable the PipeWire services for your user:

```bash
systemctl --user enable pipewire.service pipewire-pulse.service wireplumber.service
systemctl --user start pipewire.service pipewire-pulse.service wireplumber.service
```

## Firefox - Keepass

```bash
sudo pacman -Syu
sudo pacman -Sy firefox keepassxc 
```

## dotfiles

My dotfiles can be installed from [here](https://github.com/choetech160/dotfiles).  If you have installed the above hyprland config, just nvim would be needed.

```bash
sudo pacman -S nodejs npm # otherwise mason won't work
```
