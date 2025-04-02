# Arch Linux Installation

This section offers a detailed, step-by-step walkthrough for installing Arch Linux,

## Boot the Live Environment

1. Insert the USB flash drive containing the Arch Linux installation medium and restart
your computer.

2. Access the BIOS/UEFI settings (usually by pressing <kbd>DEL</kbd> or <kbd>F2</kbd>
during boot).

3. Navigate to the **Boot Menu** (often accessible via <kbd>F8</kbd>) and select the
USB flash drive containing the Arch Linux installation medium. The computer will reboot
automatically.

4. Upon reboot, the installation medium's boot loader menu will appear. Choose the
**Arch Linux install medium (x86_64, UEFI)** option, and press <kbd>Enter</kbd> to
proceed to the installation environment.

5. You'll then be logged into the first virtual console as the root user, greeted by a
Zsh shell prompt.

## Set the Console Keyboard Layout

The live environment defaults to a US keyboard layout. Available layouts can be listed with: 
```bash
# localectl list-keymaps
```
I have US keyboard layout. 

```bash
# loadkeys us
```

Console fonts are located in `/usr/share/kbd/consolefonts/` and can likewise be set with `setfont` omitting the path and file extension. For example, to use one of the largest fonts suitable for HiDPI screens, run: 

```bash
# setfont ter-132b
```

## Establishing Internet Connection

Ensure that you are connected to the internet for downloading necessary packages.

### Wired Connection During Installation

If you are using a wired connection, you should already be connected to the internet. I have wired connection so I'm skipping Wi-Fi connection steps.

## Verify the boot mode
To verify the boot mode, check the UEFI bitness:
```bash
# cat /sys/firmware/efi/fw_platform_size
```
   
If the command returns `64`, the system is booted in UEFI mode and has a 64-bit x64 UEFI.

If the command returns `32`, the system is booted in UEFI mode and has a 32-bit IA32 UEFI. While this is supported, it will limit the boot loader choice to those that support mixed mode booting.

If it returns `No such file or directory`, the system may be booted in BIOS (or CSM) mode.

If the system did not boot in the mode you desired (UEFI vs BIOS), refer to your motherboard's manual.

## Update the system clock

In the live environment `systemd-timesyncd` is enabled by default and time will be synced automatically once a connection to the internet is established.

Use `timedatectl` to ensure the system clock is synchronized:

```bash
# timedatectl
```

## Partition the Disks


Before installing Arch Linux, you need to partition the disks.

When recognized by the live system, disks are assigned to a block device such as
`/dev/sda` or `/dev/nvme0n1`. To identify these devices, use `lsblk` or `fdisk`:

```bash
# lsblk -p
```

The output should be similar to the following:

```bash
NAME             MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
/dev/sda           8:0    0 115.2G  0 disk
└─/dev/sda1        8:1    0 115.2G  0 part
/dev/nvme0n1     259:0    0   1.8T  0 disk
├─/dev/nvme0n1p1 259:1    0     2G  0 part
├─/dev/nvme0n1p2 259:2    0    64G  0 part
└─/dev/nvme0n1p3 259:3    0   1.8T  0 part
```
> [!NOTE]
> If the disk does not show up, make sure the disk controller is not in RAID mode.

`/dev/nvme0n1` is the main drive for me.

### Clean Up the Disk and Create New Partitions

Let's clean up our main drive before we create new partitions for our installation.

1. **Launch Partition Tool:**

    ```bash
    fdisk /dev/nvme0n1
    ```

2. **Initialize GPT:**

    Press <kbd>g</kbd> and then <kbd>Enter</kbd> to
    **create a new empty GPT partition table**.

3. **Create EFI Boot Partition:**

    - Press <kbd>n</kbd> and then <kbd>Enter</kbd> to **add a new partition**.

    - Press <kbd>Enter</kbd> to select the default option for the partition number.

    - Press <kbd>Enter</kbd> to select the default option for the first sector.

    - Type `+1G` and press <kbd>Enter</kbd> when it asks you for the last sector. This
    will determine the boot partition size. The Arch wiki recommends at least
    **1 GB** for the boot size. So We'll make it **1 GB**

    - Press <kbd>Y</kbd> and then <kbd>Enter</kbd> if it warns you that the partition
    contains a `vfat` signature. This will remove it.

    - Press <kbd>t</kbd> and then <kbd>Enter</kbd> to **change a partition type**.

    - Type `1` and then <kbd>Enter</kbd> to set the partition type to **EFI System**.


4. **Create Root Partition:**

    - Press <kbd>n</kbd> and then <kbd>Enter</kbd> to **add a new partition**.

    - Press <kbd>Enter</kbd> to select the default option for the partition number.

    - Press <kbd>Enter</kbd> to select the default option for the first sector.

    - Press <kbd>Enter</kbd> to select the default option for the last sector. This
    will allocate the rest of the disk to the root partition.

    - Press <kbd>Y</kbd> and then <kbd>Enter</kbd> if it warns you that the partition
    contains an `btrfs` signature. This will remove it.

    - Press <kbd>t</kbd> and then <kbd>Enter</kbd> to **change a partition type**.

    - Type `3` and then <kbd>Enter</kbd> to select the root partition.

    - Type `20` and then <kbd>Enter</kbd> to set the partition type to
    **Linux filesystem**.

5. **Write Changes and Exit:**

Press <kbd>w</kbd> and then <kbd>Enter</kbd> to **write table to disk and exit**.
Now we are done partitioning the disk.

6. **Create Zram Swap**

A simple size to start with is quarter of the total system memory. 

```bash
modprobe zram
   ```
```bash
zramctl /dev/zram0 --algorithm zstd --size "$(($(grep -Po 'MemTotal:\s*\K\d+' /proc/meminfo)/4))KiB"
```
```bash
mkswap -U clear /dev/zram0
```

```bash
swapon --discard --priority 100 /dev/zram0
```

### Verifying the Partitions

To check the partitions we created, use `fdisk`.

```bash
# fdisk -l
```

The output should be similar to the following:

```bash
Disk /dev/nvme0n1: 1.82 TiB, 2000398934016 bytes, 3907029168 sectors
Disk model: 
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 57A387AC-9145-406F-B7ED-88F282ADE694

Device             Start        End    Sectors  Size Type
/dev/nvme0n1p1      2048    4196351    4194304    1G EFI System
/dev/nvme0n1p2 138414080 3907028991 3768614912  1T Linux filesystem
Disk /dev/zram0: 8 GiB, 33559674880 byte, 8193280 sector
```

**`nvme0n1`** is the main disk

**`nvme0n1p1`** is the boot partition

**`nvme0n1p2`** is the root partition

**`zram0`** is the swap partition



### Formatting Partitions

After creating the partitions, we need to format them with a file system.

1. **EFI System Partition:**

    Format the boot partition `/dev/nvme0n1p1` to FAT32:

    ```bash
    # mkfs.fat -F 32 /dev/nvme0n1p1
    ```

    This will be our `/boot`.


2. **Root Partition:**

    Format the root partition `/dev/nvme0n1p2` as BTRFS.

    ```bash
    # mkfs.btrfs /dev/nvme0n1p2
    ```

    This will be our `/`.
   
3. **Encrypt system partition** (OPTIONAL)

```bash
# cryptsetup luksFormat /dev/nvme0n1p2

# cryptsetup luksAddKey /dev/nvme0n1p2
```
> [!TIP]
> If the header of a LUKS encrypted partition gets destroyed, you will not be able to decrypt your data. It is just as much of a dilemma as forgetting the passphrase or damaging a key-file used to unlock the partition. Damage may occur by your own fault while re-partitioning the disk later or by third-party programs misinterpreting the partition table. Therefore, having a backup of the header and storing it on another disk might be a good idea.

```bash
# cryptsetup luksHeaderBackup /dev/nvme0n1p2 --header-backup-file /mnt/<backup>/<file>.img
```
Open the encrypted system partition

```bash
# cryptsetup open /dev/nvme0n1p2
```


### Mounting Filesystems

Before installation, you must mount the newly formatted partitions.

1. **Root Partition:**

    Mount the root partition `/dev/nvme0n1p2` to `/mnt`:

    ```bash
    # mount /dev/nvme0n1p2 /mnt
    ```

2. **EFI System Partition:**

    Create a mount point for the boot partition `/dev/nvme0n1p1` at `/mnt/boot` and
    mount it:

    ```bash
    # mount --mkdir /dev/nvme0n1p1 /mnt/boot
    ```

3. **Create BTRFS Subvols**

```bash
# btrfs subvolume create /mnt/home
# btrfs subvolume create /mnt/srv
# btrfs subvolume create /mnt/var
# btrfs subvolume create /mnt/var/log
# btrfs subvolume create /mnt/var/cache
# btrfs subvolume create /mnt/var/tmp
```

## Add CachyOS repositories
Get archive with script

```bash
curl -O https://mirror.cachyos.org/cachyos-repo.tar.xz
```

Extract and enter into the archive

```bash
tar xvf cachyos-repo.tar.xz && cd cachyos-repo
```

Run script with sudo

```bash
sudo ./cachyos-repo.sh
```

## Base System Installation

Before install we need to optimize our pacman mirrorlist with "reflector".

```bash
# reflector --country TR --age 24 --protocol https --sort rate --save /etc/pacman.d/mirrorlist
# pacstrap -K /mnt base base-devel linux-cachyos-hardened linux-cachyos-hardened-headers linux-cachyos-hardened-nvidia-open nvidia-utils lib32-nvidia-utils linux-firmware nano cryptsetup btrfs-progs dosfstools util-linux git unzip sbctl kitty cachyos-settings intel-ucode
```
## Check Unified Kernel

```bash
# echo "quiet rw" >/mnt/etc/kernel/cmdline
# mkdir -p /mnt/efi/EFI/Linux
```
We need to change the HOOKS in mkinitcpio.conf to use systemd, so add it on Modules section.

`/mnt/etc/mkinitcpio.conf`

> [!NOTE]
> The "Modules" section can be seen differently. Don't mind

```bash
MODULES=(crc32c-intel nvidia nvidia_modeset nvidia_uvm nvidia_drm)
BINARIES=()
FILES=()
HOOKS=( ... .... systemd ... .. .... )

```

And now let's update the .preset file, to generate a UKI:

`/mnt/etc/mkinitcpio.d/linux-cachyos.preset`

```bash
mkinitcpio preset file for the 'linux-cachyos' package

ALL_config="/etc/mkinitcpio.conf"
ALL_kver="/boot/vmlinuz-linux-cachyos"
ALL_microcode=(/boot/intel-ucode.img)


PRESETS=('default' 'fallback')

#default_config="/etc/mkinitcpio.conf"
#default_image="/boot/initramfs-linux-cachyos.img"
default_uki="/efi/EFI/Linux/arch-linux-cachyos.efi"
default_options="--splash /usr/share/systemd/bootctl/splash-arch.bmp"

#fallback_config="/etc/mkinitcpio.conf"
#fallback_image="/boot/initramfs-linux-cachyos-fallback.img"
fallback_uki="/efi/EFI/Linux/arch-linux-cachyos-fallback.efi"
fallback_options="-S autodetect"

```
And now let's generate our UKIs:
```bash
# arch-chroot /mnt mkinitcpio -P
```
In the end it says:
`==> Unified kernel image generation successful`

If we have a look into our EFI partition, we should see our UKIs:

```bash
# ls -lR /mnt/efi
```
## Services and Boot Loader

OK, we're just about done in the archiso, we just need to enable some services, and install our bootloader:

```bash
# systemctl --root /mnt enable systemd-resolved systemd-timesyncd NetworkManager
# systemctl --root /mnt mask systemd-networkd
# arch-chroot /mnt bootctl install --esp-path=/efi
# sync
```

## Secure Boot with TPM2 Unlocking
 
 
```bash
# pacman -Syu grub efibootmgr
# sudo grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=cachyos --modules="tpm" --disable-shim-lock
# systemctl reboot --firmware-setup
```
Generate the main GRUB configuration file:

    ```bash
    # grub-mkconfig -o /boot/grub/grub.cfg
    ```
    
```bash
# sbctl status
Installed:  ✓ sbctl is installed
Setup Mode: ✗ Enabled
Secure Boot:    ✗ Disabled
Vendor Keys:    none

```
Looks good. Let's first create and enroll our personal Secure Boot keys:

```bash
# sudo sbctl create-keys
# sudo sbctl enroll-keys -m
# sudo sbctl status
# sudo sbctl verify
# sudo sbctl-batch-sign
# sudo sbctl verify
```
sbctl should now be installed and we can proceed to signing the kernel images and boot manager.

Let's reinstall the kernel to make sure it resigns the UKI:

```bash
# sudo pacman -S linux-cachyos-hardened

Signing EFI binaries...
Generating EFI bundles....
✓ Signed /efi/EFI/Linux/arch-linux-cachyos-fallback.efi
✓ Signed /efi/EFI/Linux/arch-linux-cachyos.efi
```

Looking good. Reboot your PC now, so the Secure Boot settings will get saved. Once rebooted we need to configure automatic unlocking of the root filesystem, by binding a LUKS key to the TPM. Let's generate a new key, add it to our volume so it can be used to unlock it in addition to the existing keys, and bind this new key to PCRs 0 and 7 (the system firmware and Secure Boot state). First things first, let's generate a recovery key in case it all gets messed up some time in the future:
```bash
# sudo systemd-cryptenroll /dev/gpt-auto-root-luks --recovery-key
# sudo systemd-cryptenroll --tpm2-device=auto --tpm2-pcrs=0+7  /dev/gpt-auto-root-luks
```

## Generating the fstab

Generate the fstab (file system table) configuration file to define how disk
partitions, block devices, or remote file systems are mounted into the filesystem:

```bash
# genfstab -U /mnt >> /mnt/etc/fstab
```

## Change Root (chroot)

Switch to the root environment of your newly installed system:

```bash
# arch-chroot /mnt
```

This step allows you to execute commands as if your new system were already running.

## Essential Package Installation

Now we are ready to install the essential packages for Arch Linux.

1. **NetworkManager:**

    `NetworkManager` allows you to configure and manage network connections.

    ```bash
    # pacman -Syu networkmanager
    ```

    You can use it to connect to Wi-Fi or Ethernet networks after completing the Arch
    Linux installation.

2. **Vim:**

    Vim is a powerful terminal text editor that you can use to modify text files.

    ```bash
    # pacman -Syu gvim
    ```

    GVim is essentially the same package as Vim with GTK/X support.

3. **Sudo:**

    Sudo allows specified users to execute commands as the root user or another user,
    as specified in the `sudoers` file, enhancing security and control.

    ```bash
    # pacman -Syu sudo
    ```

## Configuring Users and Groups

1. **Root Password:**

    Ensure the root account has a secure password:

    ```bash
    # passwd
    ```

2. **Creating a New User:**

    For everyday tasks, it's best to use a non-root user.

    - I use `skyline` as the username for the new user account:

        ```bash
        # useradd -m skyline
        ```

    - Set the password for the new user account:

        ```bash
        # passwd skyline
        ```

    - Add the new user account to the `wheel` group:

        ```bash
        # gpasswd -a skyline wheel
        ```

        The `wheel` group is the administration group and is commonly used to grant
        administrative privileges.

3. **Configuring Sudo:**

    Edit the `sudoers` file to grant `wheel` group members sudo privileges:

    ```bash
    # EDITOR=vim visudo
    ```

    Uncomment the line (remove #):

    ```properties
    # %wheel ALL=(ALL:ALL) ALL
    ```

    Save the file and exit the editor.


## Network Configuration

To be able to connect to the internet on next boot, enable NetworkManager service:

```bash
# systemctl enable NetworkManager.service
```

## Regenerating Initramfs

Refresh the initial ramdisk setup to incorporate all installed modules and
configurations:

```bash
# mkinitcpio -P
```

## Final Steps: Exiting chroot and Rebooting

1. Exit the chroot environment with `exit` or <kbd>Ctrl</kbd> + <kbd>d</kbd>.

2. Reboot your system with `reboot`.

Remember to remove the installation media to boot into your new Arch Linux setup.

### First Boot into Arch Linux

1. When the GRUB bootloader menu appears, select **Arch Linux** and press
<kbd>Enter</kbd> to boot into Arch Linux.

2. When prompted, log in with the new user account you created.

## Table of Contents

### [1. Home](./README.md)

### [2. Installation Guide](./INSTALLATION.md)

### [3. First Steps](./FIRSTSTEPS.md)

### [4. Graphical User Interface](./GUI.md)

### [5. Optional Tools and Configurations](./OPTIONAL.md)

### [6. Additional Information](./APPENDIX.md)

### [7. License](./LICENSE)
