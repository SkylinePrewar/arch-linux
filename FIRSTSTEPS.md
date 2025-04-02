# First Steps After Installing Arch Linux

## Setting Up Localization

The `locale` defines the language used by the system, as well as other regional
considerations such as currency denomination, numerology, and character sets.

1. **Enable Required Locales:**

    Uncomment the lines for German, English, Turkish and Vietnamese locales in
    `/etc/locale.gen`:

    ```bash
    sudo sed -i -e 's/#\(de_CH\.UTF-8 UTF-8\)/\1/' \
    -e 's/#\(de_CH ISO-8859-1\)/\1/' \
    -e 's/#\(en_US\.UTF-8 UTF-8\)/\1/' \
    -e 's/#\(en_US ISO-8859-1\)/\1/' \
    -e 's/#\(tr_TR\.UTF-8 UTF-8\)/\1/' \
    -e 's/#\(tr_TR ISO-8859-9\)/\1/' \
    -e 's/#\(vi_VN UTF-8\)/\1/' /etc/locale.gen
    ```

2. **Verify Locales:**

    Check the contents of the `/etc/locale.gen` file and correct any errors:

    ```bash
    sudo vim /etc/locale.gen
    ```

3. **Generate Locales:**

    Apply the changes by generating the locales:

    ```bash
    sudo locale-gen
    ```

4. **Set System Language and Keyboard:**

    - Define U.S. English as the default system language:

        ```bash
        sudo localectl set-locale LANG=en_US.UTF-8
        ```

    - Configure the console and X11 keyboard layouts to Swiss German:

        ```bash
        sudo localectl set-keymap --no-convert de_CH-latin1
        sudo localectl set-x11-keymap --no-convert ch
        ```

5. **Review Localization Settings:**

   Confirm your configurations are correctly applied:

    ```bash
    sudo localectl status
    ```

6. **Restart Computer:**

    Reboot the computer to apply the changes.

## Display and Keyboard Configuration


## Enable Arch User Repository (AUR)

The Arch User Repository (AUR) offers a vast collection of community-maintained
packages, expanding the software available for Arch Linux.

1. **Install Essential Build Tools:**

    The `base-devel` group contains tools necessary for compiling AUR packages.

    ```bash
    sudo pacman -Syu base-devel
    ```

2. **Prepare AUR Packages Directory:**

    Create a dedicated directory for managing AUR package builds:

    ```bash
    mkdir -p ~/.aur
    ```

## Set Up Yay

```bash
# git clone https://aur.archlinux.org/yay-bin.git
# cd yay-bin
# makepkg -si
```

First use:

    Use `yay -Y --gendb` to generate a development package database for *-git packages that were installed without yay. This command should only be run once.

    `yay -Syu --devel` will then check for development package updates

    Use `yay -Y --devel --save` to make development package updates permanently enabled (yay and yay -Syu will then always check dev packages)


## Zen Browser

Zen Browser is a free and open-source fork of Mozilla Firefox, with its main focus being privacy, customizability and design, and it is licensed under the Mozilla Public License 2.0 (MPL 2.0)

### Installation

Ensure you have the latest version of Zen Browser:

```bash
yay zen-browser
```

During the installation process, if prompted to select a provider for ttf-font,
choose `noto-fonts`.

### Configuration

1. **Set Up User.Js:**

Use with Arkenfox's or Yokoffing's User.js
    

2. **Enable Bookmarks Toolbar:**

    - Right-click on the menu button (three horizontal lines) in the top-right corner.

    - Select **Bookmarks Toolbar > Always Show**.

    - Right-click on **Getting Started** bookmark and select **Delete Bookmark**.

3. **Pin Extensions to Toolbar:**

    - Click on the extensions menu button in the top-right corner.

    - Right-click on **uBlock Origin** and select **Pin to Toolbar**.

    - Right-click on any other extensions and unpin them by deselecting
    **Pin to Toolbar**.

4. **Run Extensions in Private Windows:**

    - Click on the extensions menu button in the top-right corner.

    - Select **Manage Extensions**.

    - Allow for all:
    **Run in Private Windows**.

5. **Activate and Configure Extensions:**

    - uBlock Origin, Dark Reader, KeePassXC, LibRedirect.

6. **Fine-tuning**
   
   - Catpuccin Themes and Zen mods.
   - Compact Mode.


## Checking System Clock Synchronization

It's crucial to ensure that the system clock is accurately synchronized to avoid issues
with time-based applications and services.

1. To set your time zone:

    ```bash
    timedatectl set-timezone Europe/Istanbul
    ```

2. Install the `ntp` package to synchronize the system clock:

    ```bash
    sudo pacman -Syu ntp
    ```

3. Perform a one-time synchronization by starting `ntpd` from the console using the
following command:

    ```bash
    sudo ntpd -u ntp:ntp
    ```

4. To maintain synchronization, enable the `ntpd.service` to start automatically at
boot:

    ```bash
    systemctl enable ntpd.service
    ```

5. Set the hardware clock from the system clock:

    ```bash
    sudo hwclock --systohc
    ```

6. Use `ntpq` to see the list of configured peers and status of synchronization:

    ```bash
    ntpq -p
    ```

    The delay, offset and jitter columns should be non-zero. The servers `ntpd` is
    synchronizing with are prefixed by an asterisk.

    > :warning: **Note**\
    > It can take several minutes before `ntpd` selects a server to synchronize with.
    > Try checking after 17 minutes (1024 seconds).

## Network Configuration

### How to Set the Hostname on Your System

To set the hostname on your system, please follow these steps:

1. Create the `/etc/hostname` file and add your desired hostname to it. For
instance, if you want to set the hostname as `bahadir-desktop`, then run the
following command:

    ```bash
    echo "skyline" | sudo tee /etc/hostname > /dev/null
    ```

2. Verify that the `/etc/hostname` file contains the correct hostname by running
the following command:

    ```bash
    sudo vim /etc/hostname
    ```

    Ensure that the file contains only the desired hostname as shown below:

    ```properties
    sykline
    ```

### How to Resolve Local Network Hostnames

1. Run the following command to add the necessary lines to `/etc/hosts`. This
file maps hostnames to IP addresses:

    ```bash
    echo -e "\
    # Static table lookup for hostnames.\n\
    # See hosts(5) for details.\n\n\
    # The following lines are desirable for IPv4 capable hosts\n\
    127.0.0.1       localhost\n\
    # 127.0.1.1 is often used for the FQDN of the machine\n\
    127.0.1.1       skyline\n\n\
    # The following lines are desirable for IPv6 capable hosts\n\
    ::1             localhost\n\
    " | sudo tee /etc/hosts > /dev/null
    ```

2. Verify that the `/etc/hosts` file contains the correct entries by running the
following command:

    ```bash
    sudo vim /etc/hosts
    ```

    Ensure that the file contains the following entries:

    ```properties
    # The following lines are desirable for IPv4 capable hosts
    127.0.0.1       localhost
    # 127.0.1.1 is often used for the FQDN of the machine
    127.0.1.1       skyline

    # The following lines are desirable for IPv6 capable hosts
    ::1             localhost
    ```

    The `127.0.1.1` entry maps the hostname to the loopback address. If you have
    a static IP address, use it instead of `127.0.1.1`.

## Pacman Configuration

Pacman is the default package manager for Arch Linux, which is used to install, remove,
and update software packages in the system.

### Adding Multilib Repository

To add the multilib repository, follow these steps:

1. Uncomment the `multilib` section in the `/etc/pacman.conf` file by using the
following command:

    ```bash
    sudo sed -i -e '/^#\?\[multilib]/,/^#\?\(\[[^multilib]\|$\)/ {
    /^#\?\[[^multilib]/d
    s/^#//
    }' /etc/pacman.conf
    ```

2. Check the contents of `/etc/pacman.conf` file for any errors by using the following
command:

    ```bash
    sudo micro /etc/pacman.conf
    ```

    The `multilib` section should look like this:

    ```properties
    [multilib]
    Include = /etc/pacman.d/mirrorlist
    ```

### Miscellaneous Options

To enable colors and parallel downloads, follow these steps:

1. Use the following command:

    ```bash
    sudo sed -i -e '/^# Misc options$/,/^$/ {
    /^#Color/s/^#//
    /^#ParallelDownloads = 5/s/^#//
    }' /etc/pacman.conf
    ```

2. Check the contents of `/etc/pacman.conf` file for any errors by using the following
command:

    ```bash
    sudo micro /etc/pacman.conf
    ```

    The # Misc options section should look like this:

    ```properties
    # Misc options
    #UseSyslog
    Color
    #NoProgressBar
    CheckSpace
    #VerbosePkgLists
    ParallelDownloads = 5
    ```

### Update Repositories and Packages

To update repositories and packages, use the following command:

```bash
sudo pacman -Syu
```

If updating returns an error, open the `pacman.conf` file again and check for errors.

### Timeshift and Grub-BTRFS

Install `timeshift` and `grub-btrfs` packages.

```bash
yay timeshift grub-btrfs
```
Open Timeshift with `sudo -E timeshift-gtk` command on terminal.

Check for instructions.

`https://github.com/Antynea/grub-btrfs`

### Clean Pacman Cache and Timeshift Autosnap

Periodically cleaning up the cache is necessary to prevent the directory from growing
indefinitely in size. To clean the cache, follow these steps:

1. Install the `pacman-contrib` package by using the following command:

    ```bash
    sudo pacman -Syu pacman-contrib
    ```

    The `pacman-contrib` package includes the `paccache` script, which deletes all
    cached versions of installed and uninstalled packages, except for the most recent
    three, by default.

2. Enable and start `paccache.timer` to discard unused packages weekly by using the
following command:

    ```bash
    systemctl enable --now paccache.timer
    ```

3. Install the `timeshift-autosnap`
   Timeshift auto-snapshot script which runs before package upgrade using Pacman hook.
   
  ```bash
   #  yay timeshift-autosnap
   ```
`/etc/timeshift-autosnap.conf`

    skipAutosnap - if set to true script won't be executed.
    deleteSnapshots - if set to false old snapshots won't be deleted.
    maxSnapshots - defines maximum number of old snapshots to keep.
    updateGrub - if set to false grub entries won't be generated.
    snapshotDescription - defines value used to distinguish snapshots created using timeshift-autosnap.



## Table of Contents

### [1. Home](./README.md)

### [2. Installation Guide](./INSTALLATION.md)

### [3. First Steps](./FIRSTSTEPS.md)

### [4. Graphical User Interface](./GUI.md)

### [5. Optional Tools and Configurations](./OPTIONAL.md)

### [6. Additional Information](./APPENDIX.md)

### [7. License](./LICENSE)
