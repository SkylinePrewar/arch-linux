# Skyline's Personal Arch Linux Installation Guide

This guide is a personalized, comprehensive Arch Linux installation tutorial for myself,
Skyline, to guide me through the steps needed to build a customized Arch Linux system.

Arch Linux is a lightweight Linux distribution, providing a bare minimum base system.
This minimalist design allows users to include only the components they need, making it
remarkably fast, reliable, and ideal for those who want an extensive control over their
operating environment.

With clear step-by-step instructions, you'll be able to navigate through the
installation with ease and confidence.

## Hardware Configuration

- **Processor:** Intel® Core™ i5-9400F 9MB Cache, 2.90 GHz up to 4.10 GHz ( 6 Threads, 6 Core )

- **Graphics Card:** NVIDIA GeForce RTX™ 2060 (6 GB)

- **Storage Drive:** Samsung 970 EVO Plus (1000 GB, M.2 2280), Kingston A400 (480 GB, SATA SSD)

- **Memory Kit:** Corsair VENGEANCE® LPX 32 GB (4 X 8 GB, 3200 MHz, DDR4 DRAM, DIMM)

- **Motherboard:** Asus Prime Z390-P ( LGA 1151, Intel Z390, ATX)


## Obtain and Verify the Installation Image

To start, follow these steps to obtain and authenticate the installation image:

1. Change the directory to `Downloads`:

    ```bash
    cd ~/Downloads
    ```

2. Access the [Arch Linux Downloads](https://www.archlinux.org/download/) page and
select a mirror to download the `.iso` and `.sig` files.

3. Set your mirror and file names as environment variables:

    ```bash
    export MIRROR_SITE="https://geo.mirror.pkgbuild.com/iso/latest"
    export ISO_FILE="archlinux-x86_64.iso"
    export SIG_FILE="${ISO_FILE}.sig"
    ```

4. Download the files with `curl`:

    ```bash
    curl -O "${MIRROR_SITE}/${ISO_FILE}"
    curl -O "${MIRROR_SITE}/${SIG_FILE}"
    ```

5. Verify the authenticity of the installation image:

    ```bash
    gpg --keyserver-options auto-key-retrieve --verify "${SIG_FILE}"
    ```

    Look for a message in the output confirming the signature's validity and the
    trusted status of the key, similar to this:

    ```bash
    gpg: assuming signed data in 'archlinux-x86_64.iso'
    gpg: Signature made Mon 01 Jan 2025 05:48:11 PM CET
    gpg:                using EDDSA key 3E80CA1A8B89F69CBA57D98A76A5EF9054449A5C
    gpg:                issuer "pierre@archlinux.org"
    gpg: key 76A5EF9054449A5C: public key "Pierre Schmitz <pierre@archlinux.org>" imported
    gpg: key 7F2D434B9741E8AC: public key "Pierre Schmitz <pierre@archlinux.org>" imported
    gpg: Total number processed: 2
    gpg:               imported: 2
    gpg: no ultimately trusted keys found
    gpg: Good signature from "Pierre Schmitz <pierre@archlinux.org>" [unknown]
    gpg: WARNING: The key's User ID is not certified with a trusted signature!
    gpg:          There is no indication that the signature belongs to the owner.
    Primary key fingerprint: 3E80 CA1A 8B89 F69C BA57  D98A 76A5 EF90 5444 9A5C
    ```

    Ensure the fingerprint in the output matches the [Master Key Signatures](https://archlinux.org/master-keys/#master-sigs).

## Create a USB Flash Installation Medium

A bootable USB drive is essential for installing Arch Linux:

1. Use `lsblk` to identify the name of the USB flash drive, and ensure it is not
mounted:

    ```bash
    lsblk
    ```

    The output should display the name of the USB drive, similar to the example below:

    ```bash
    NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
    sda           8:0    0  480G  0 disk
    └─sda1        8:1    0  480G  0 part
    sdb           8:16   1  58.6G  0 disk
    ├─sdb1        8:17   1   798M  0 part
    └─sdb2        8:18   1    15M  0 part
    nvme0n1     259:0    0    1T 0 disk
    ├─nvme0n1p1 259:1    0     2G  0 part /boot
    ├─nvme0n1p2 259:2    0    64G  0 part [SWAP]
    └─nvme0n1p3 259:3    0    1T  0 part /
    ```

    In my case, the USB drive name is `/dev/sdb`.

2. Write the installation image to the USB drive:

    ```bash
    export USB_DEV="/dev/sdx" # Replace 'x' with your USB drive letter.
    sudo dd bs=4M if="${ISO_FILE}" of="${USB_DEV}" conv=fsync oflag=direct status=progress
    sudo sync
    ```

    > ⚠️ **Warning:**\
    > This will permanently erase all data on `/dev/sdx` and write the installation
    image onto it. Ensure you have entered the correct drive name.

## Next Steps

With the bootable USB ready, you're set to install Arch Linux. Follow the links below
in sequence to customize and optimize your Arch Linux setup:

### [1. Home](./README.md)

### [2. Installation Guide](./INSTALLATION.md)

### [3. First Steps](./FIRSTSTEPS.md)

### [4. Graphical User Interface](./GUI.md)

### [5. Optional Tools and Configurations](./OPTIONAL.md)

### [6. Additional Information](./APPENDIX.md)

### [7. License](./LICENSE)
