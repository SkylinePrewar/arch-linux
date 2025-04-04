# Additional Information

## System and Package Management

### Upgrading the System and Packages

Arch Linux is a rolling release distribution, which means that its software packages
are frequently updated to ensure that users have the latest features, bug fixes, and
security patches.

1. To update all packages on the system, use the following command to synchronize the
repository databases and upgrade the system:

    ```bash
    sudo pacman -Syu
    ```

    This command could take some time depending on how up-to-date the system is.

2. Once the upgrade is complete, restart your system to ensure that all upgrades are
applied to existing processes.



### Upgrading the AUR Packages

Arch Linux's community repository (AUR) contains a vast number of user-contributed
packages that are not part of the official Arch Linux repositories. To update AUR
packages, you must first update the files and changes using the `git pull` command in
the directory containing the package's `PKGBUILD`. Then, build and install the package.

1. The following command updates each package inside the `~/.aur` directory:

    ```bash
    find ~/.aur -mindepth 1 -maxdepth 1 -type d -exec sh -c 'cd "{}" && git pull && makepkg -sirc --noconfirm && git clean -dfx' \;
    ```

    This command uses the find command to locate all the subdirectories of `~/.aur`, and
    then executes the `git pull`, `makepkg -sirc`, and `git clean -dfx` commands in
    each subdirectory.

2. Once the upgrade is complete, restart your system to ensure that all upgrades are
applied to existing processes.

### System Upgrade After a Prolonged Interval

In cases where system updates are delayed significantly, follow these steps to ensure
system integrity.

First, manually synchronize the package database and update the `archlinux-keyring`
package, then proceed to a full system upgrade using the following command:

```bash
sudo pacman -Sy archlinux-keyring && sudo pacman -Su
```

This is not considered a partial upgrade as it synchronizes the package database and
upgrades the keyring package first, ensuring all packagevsignatures can be properly
verified during the system upgrade.

### Uninstalling Existing Packages

To remove a package along with its dependencies that are not required by any other
installed package, use the following command:

```bash
sudo pacman -Rsu <PACKAGE_NAME>
```

Replace `<PACKAGE_NAME>` with the name of the package you want to uninstall.

## Managing USB Devices

### Query Connected USB Devices

1. The `lsusb` tool is available inside the `usbutils` package:

    ```bash
    sudo pacman -S usbutils
    ```

2. Run the `lsusb` command to display a list of all connected USB devices:

    ```bash
    lsusb
    ```

    The result should look like this:

    ```bash
    Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
    Bus 001 Device 002: ID 0b05:19af ASUSTek Computer, Inc. [unknown]
    Bus 001 Device 003: ID 05e3:0608 Genesys Logic, Inc. Hub
    Bus 001 Device 004: ID 2188:5802 No brand [unknown]
    Bus 001 Device 005: ID 8087:0033 Intel Corp. [unknown]
    Bus 001 Device 006: ID 0451:ace1 Texas Instruments, Inc. [unknown]
    Bus 001 Device 007: ID 2188:5510 No brand [unknown]
    Bus 001 Device 008: ID 2188:7112 No brand [unknown]
    Bus 001 Device 009: ID 2188:5511 No brand [unknown]
    Bus 001 Device 010: ID 2188:5512 No brand [unknown]
    Bus 001 Device 011: ID 05e3:0608 Genesys Logic, Inc. Hub
    Bus 001 Device 012: ID 046d:c52b Logitech, Inc. Unifying Receiver
    Bus 001 Device 013: ID 0b05:1a7a ASUSTek Computer, Inc. [unknown]
    Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
    Bus 002 Device 002: ID 8564:1000 Transcend Information, Inc. JetFlash
    Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
    Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
    ```

### Safely Ejecting a USB Flash Drive

1. Verify that the USB flash drive is not mounted. Run the following command to display
a list of all connected disks and their partitions:

    ```bash
    lsblk
    ```

    Look for the USB flash drive and its partitions in the output:

    ```bash
    NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
    sda           8:0    0 931.5G  0 disk
    └─sda1        8:1    0 931.5G  0 part
    sdb           8:16   1  58.6G  0 disk
    ├─sdb1        8:17   1   809M  0 part
    └─sdb2        8:18   1    15M  0 part
    nvme0n1     259:0    0 953.9G  0 disk
    ├─nvme0n1p1 259:1    0     1G  0 part /boot
    ├─nvme0n1p2 259:2    0    32G  0 part [SWAP]
    └─nvme0n1p3 259:3    0 920.9G  0 part /
    ```

    For example, if the USB flash drive is `/dev/sdb`, you should see `/dev/sdb1` and
    `/dev/sdb2`.

2. Install the `udisks2` package, which provides a tool to query and manipulate storage
devices:

    ```bash
    sudo pacman -S udisks2
    ```

3. If any partitions on the USB flash drive are mounted, unmount them with the
`udisksctl unmount` command:

    ```bash
    udisksctl unmount -b /dev/sdb1
    ```

    This will ensure that the partitions on the USB flash drive are not in use.

4. Optionally, you can use the `fdisk` command to erase the partition table and create
a new one. Be careful with this command, as it will erase all data on the USB flash
drive. Run the following command to start `fdisk`:

    ```bash
    sudo fdisk /dev/sdb
    ```

    - Press <kbd>o</kbd> and then <kbd>Enter</kbd> to create a new empty DOS partition
    table.

    - Press <kbd>n</kbd> and then <kbd>Enter</kbd> to add a new partition.

    - Follow the prompts to create new partitions.

    - Press <kbd>w</kbd> and then <kbd>Enter</kbd> to write the changes to the USB
    flash drive and exit.

    - Format `/dev/sdb1` partition as `EXT4`:

    ```bash
    sudo mkfs.ext4 /dev/sdb1
    ```

5. Power off the USB flash drive and remove it from the computer using the `udisksctl`
command:

    ```bash
    udisksctl power-off -b /dev/sdb
    ```

    This will safely power off the USB flash drive and allow you to remove it from the
    computer.

### Retrieving Images and Videos from Android Devices

1. Connect your Android device to your computer using a USB cable. Ensure that
your device is unlocked, and if needed, allow USB debugging for the device to
be recognized.

2. After successfully connecting the device, swipe down from the top of your
Android device's screen to open the notification panel. Locate and tap on the
**USB connection** notification, then select the **File Transfer** or
**MTP (Media Transfer Protocol)** mode. This will allow your computer to access
the files stored on the Android device.

3. On your computer, open a file manager to navigate to the storage of your
Android device. The device will typically appear as a separate device in the
file manager. Right-click on the file manager within the Android device's
storage and select **Open terminal here**.

4. Execute the following command to search for all image and video files in
this directory and its subdirectories, and move them to `~/Pictures/Import` on
your computer:

    ```bash
    $ find . -type f \( -iname \*.jpg -o -iname \*.png -o -iname \*.jpeg -o -iname \*.gif -o -iname \*.bmp -o -iname \*.tif -o -iname \*.tiff -o -iname \*.jfif -o -iname \*.webp -o -iname \*.heif -o -iname \*.indd -o -iname \*.ai -o -iname \*.eps -o -iname \*.mp4 -o -iname \*.avi -o -iname \*.mkv -o -iname \*.flv -o -iname \*.mov -o -iname \*.wmv -o -iname \*.vob -o -iname \*.mng -o -iname \*.qt -o -iname \*.yuv -o -iname \*.rm -o -iname \*.rmvb -o -iname \*.asf -o -iname \*.amv -o -iname \*.mpg -o -iname \*.mpeg -o -iname \*.m4v -o -iname \*.svi -o -iname \*.3gp -o -iname \*.3g2 \) -exec mv -i {} ~/Pictures/Import/ \;
    ```

## Managing Network

### NextDNS and Zapret

Go to and log-in 

`https://my.nextdns.io/login`

Copy and Paste commands at `Linux` and `Browser` sections. Follow the instructions.

Install `zapret` package and watch how to install youtube videos.

## Set Up System Backup

### Boot the Live Environment

Insert the USB flash drive containing the Arch Linux installation medium and
follow the steps below:

1. Reboot or turn on your computer and access firmware (BIOS) settings by
pressing the <kbd>DEL</kbd> key during the power-on self-test (POST) phase.

2. In the **Boot** tab on the BIOS screen, set the
**FIXED BOOT ORDER Priorities** as follows:

    - Boot Option #1: [USB Hard Disk]

    - Boot Option #2: [Hard Disk]

    - Boot Option #3: [USB CD/DVD]

    - Boot Option #4: [USB Lan]

3. In the **Save & Exit** tab, click on **Save Changes and Reset**. The
computer will restart automatically.

4. When the installation medium's boot loader menu appears, select
**Arch Linux install medium (x86_64, UEFI)** and press Enter to enter the
installation environment

5. You will be logged in to the first virtual console as the root user and
presented with a Zsh shell prompt.

### Clone the Disk Using Clonezilla

Cloning your disk with Clonezilla can help you backup your system in case of
hardware failure or other issues.

1. Open Clonezilla with the following command in the terminal:

    ```bash
    # clonezilla
    ```

2. In the Clonezilla interface, choose the **device-image** option for backup.

3. Choose the **local_dev** option to save the backup image to a local device.
Clonezilla will scan the disks on the machine every few seconds and display the
results. If you haven't inserted your backup disk already, insert it now. Press
<kbd>Ctrl</kbd> + <kbd>C</kbd> to finish scanning once you see your backup disk.

4. Choose the device partition to backup: `/dev/sda1`. Press <kbd>Enter</kbd>.
The partition type must be `ext4`. If necessary, quit Clonezilla and format the
disk properly using the `fdisk` command before running Clonezilla again.

5. Choose a directory on the target device where you want to store the backup
image. The default directory `/` is fine.

6. Select the option **Beginner Mode**.

7. Select **savedisk** mode.

8. Enter the image name. Clonezilla will give an image name based on date and
time, but you can feel free to change it.

9. Choose the source disk that you want to backup: `/dev/nvme0n1`

10. Skip the file system check.

11. Say **Yes** to the option **Check the saved image** to ensure that the image
is valid.

12. Do not encrypt the image.

13. Select **choose** as the action you want to perform after the image saving
is done.

14. The pocess of cloning your disk can take a significant amount of time
depending on the size of your disk and the speed of your computer. Be patient
and wait for Clonezilla to finish creating the backup image.

15. Once the backup process is complete, you will return to the console as the
root user and presented with a Zsh shell prompt. Safely eject the backup disk
using the appropriate commands:

    ```bash
    $ udisksctl unmount -b /dev/sda1
    ```

    ```bash
    $ udisksctl power-off -b /dev/sda
    ```

16. The backup process is now complete. Keep the backup disk in a safe place
and label it properly to avoid any confusion in the future.

17. Finally, type `reboot` to restart the computer.

### Boot Back into Arch Linux

1. After reboot, access firmware (BIOS) settings by pressing the <kbd>DEL</kbd>
key during the power-on self-test (POST) phase.

2. In the **Boot** tab on the BIOS screen, set the
**FIXED BOOT ORDER Priorities** as follows:

    - Boot Option #1: [Hard Disk: GRUB]

    - Boot Option #2: [USB Hard Disk]

    - Boot Option #3: [USB CD/DVD]

    - Boot Option #4: [USB Lan]

3. In the **Save & Exit** tab, click on **Save Changes and Reset**. The
computer will restart automatically.

4. When the GRUB boot loader menu appears, select **Arch Linux** and press
<kbd>Enter</kbd> to boot into Arch Linux.

## Restore Disk From Backup Image

Once you have created a backup of your disk using Clonezilla, you can use it to
restore your system in case of a failure or data loss.

### Boot into the Live Environment

Insert the USB flash drive containing the Arch Linux installation medium and
follow the steps below:

1. Reboot or turn on your computer and access firmware (BIOS) settings by
pressing the <kbd>DEL</kbd> key during the power-on self-test (POST) phase.

2. In the **Boot** tab on the BIOS screen, set the
**FIXED BOOT ORDER Priorities** as follows:

    - Boot Option #1: [USB Hard Disk]

    - Boot Option #2: [Hard Disk]

    - Boot Option #3: [USB CD/DVD]

    - Boot Option #4: [USB Lan]

3. In the **Save & Exit** tab, click on **Save Changes and Reset**. The
computer will restart automatically.

4. When the installation medium's boot loader menu appears, select
**Arch Linux install medium (x86_64, UEFI)** and press Enter to enter the
installation environment

5. You will be logged in to the first virtual console as the root user and
presented with a Zsh shell prompt.

### Restore the Disk from Backup Image Using Clonezilla

1. Open Clonezilla with the following command in the terminal:

    ```bash
    # clonezilla
    ```

2. In the Clonezilla interface, choose the **device-image** option for backup.

3. Choose the **local_dev** option to restore the backup image from the local
device. Clonezilla will scan the disks on the machine every few seconds and
display the results. If you haven't inserted your backup disk already, insert it
now. Press <kbd>Ctrl</kbd> + <kbd>C</kbd> to finish scanning once you see your
backup disk.

4. Choose the device partition to backup: `/dev/sda1`. Press <kbd>Enter</kbd>.
The partition type must be `ext4`. If necessary, quit Clonezilla and format the
disk properly using the `fdisk` command before running Clonezilla again.

5. Choose the directory on the local device where the backup image is stored.
For instance: `/`.

6. Select the option **Beginner Mode**.

7. Select **restoredisk** mode.

8. Select the image to restore.

9. Choose the source disk to restore the backup to: `/dev/nvme0n1`

10. Say **Yes** to the option **Check the image before restoring** to know if
the image is broken or not.

11. Select **choose** as the action you want to perform after the image saving
is done.

12. The process of restoring your disk can take a significant amount of time
depending on the size of your disk and the speed of your computer. Be patient
and wait for Clonezilla to finish restoring the backup image.

13. Once the restoration process is complete, you will return to the console as
the root user and presented with a Zsh shell prompt. Safely eject the backup
disk using the appropriate commands:

    ```bash
    $ udisksctl unmount -b /dev/sda1
    ```

    ```bash
    $ udisksctl power-off -b /dev/sda
    ```

14. The restore process is now complete.

15. Finally, type `reboot` to restart the computer.

### Boot Back into Operating System

1. After reboot, access firmware (BIOS) settings by pressing the <kbd>DEL</kbd>
key during the power-on self-test (POST) phase.

2. In the **Boot** tab on the BIOS screen, set the
**FIXED BOOT ORDER Priorities** as follows:

    - Boot Option #1: [Hard Disk: GRUB]

    - Boot Option #2: [USB Hard Disk]

    - Boot Option #3: [USB CD/DVD]

    - Boot Option #4: [USB Lan]

3. In the **Save & Exit** tab, click on **Save Changes and Reset**. The
computer will restart automatically.

4. When the GRUB boot loader menu appears, select **Arch Linux** and press
<kbd>Enter</kbd> to boot into Arch Linux.

## Table of Contents

### [1. Home](./README.md)

### [2. Installation Guide](./INSTALLATION.md)

### [3. First Steps](./FIRSTSTEPS.md)

### [4. Graphical User Interface](./GUI.md)

### [5. Optional Tools and Configurations](./OPTIONAL.md)

### [6. Additional Information](./APPENDIX.md)

### [7. License](./LICENSE)
