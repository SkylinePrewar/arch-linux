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

### GNOME Session

1. **Adjust Display Scaling:**

    Improve readability and UI scaling on high-resolution displays:

    ```bash
    gsettings set org.gnome.settings-daemon.plugins.xsettings overrides "[{'Gdk/WindowScalingFactor', <2>}]"
    gsettings set org.gnome.desktop.interface scaling-factor 2
    ```

2. **Configure Keyboard Layout:**

   Set the keyboard layout to Swiss German for GNOME:

    ```bash
    gsettings set org.gnome.desktop.input-sources sources "[('xkb', 'ch')]"
    ```

3. **Set Titlebar Button Order:**

    Set the order for the GNOME window manager:

    ```bash
    gsettings set org.gnome.desktop.wm.preferences button-layout ':minimize,maximize,close'
    ```

4. **Disable "Show Notifications" Shortcut:**

    Disable the <kbd>Super</kbd> + <kbd>V</kbd> shortcut for showing notifications in
    GNOME:

    ```bash
    gsettings set org.gnome.shell.keybindings toggle-message-tray "['']"
    ```

### GNOME Display Manager (GDM)

1. **Log into GDM Session**

    Access the GDM environment:

    ```bash
    sudo machinectl shell gdm@ /bin/bash
    ```

2. **Adjust Display Scaling:**

    Ensure GDM's display scaling matches your GNOME session settings:

    ```bash
    dbus-launch gsettings set org.gnome.settings-daemon.plugins.xsettings overrides "[{'Gdk/WindowScalingFactor', <2>}]"
    dbus-launch gsettings set org.gnome.desktop.interface scaling-factor 2
    ```

3. **Restart Computer:**

    Reboot the computer to apply the changes.

### GNOME Tools and Extensions

1. **Install GNOME Tweaks:**

    GNOME implements XDG Autostart. Install `gnome-tweaks` tool to manage startup
    applications:

    ```bash
    sudo pacman -Syu gnome-tweaks
    ```

2. **Install AppIndicator Extension:**

    Install `gnome-shell-extension-appindicator` to enable system tray icons:

    ```bash
    sudo pacman -Syu gnome-shell-extension-appindicator
    ```

3. **Restart Computer:**

    Reboot the computer to apply the changes.

4. **Enable AppIndicator Extension:**

    Enable the extension containing the `"appindicatorsupport"` string:

    ```bash
    gnome-extensions enable $(gnome-extensions list | grep -m 1 appindicatorsupport)
    ```

5. **Enable System Monitor Extension:**

    Enable the extension containing the `"system-monitor"` string:

    ```bash
    gnome-extensions enable $(gnome-extensions list | grep -m 1 system-monitor)
    ```

## Command Line Enhancements

Bash is the default command-line shell on Arch Linux.

### Bash Completion

Enhance auto-completion features in Bash for a more efficient command-line experience:

```bash
sudo pacman -Syu bash-completion
```

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

## Set Up Git

Git is a widely used version control system for tracking changes to files and
directories.

### Installation

Ensure you have the latest version of Git:

```bash
sudo pacman -Syu git
```

### Organizing Git Repositories

Create a dedicated `git` folder for storing cloned repositories:

```bash
mkdir -p ~/git
```

### Configuring Git

Set your identity and preferred editor.

1. **Email and Name:**

    Personalize your Git commits by setting your name and email:

    ```bash
    git config --global user.name "Bahadır Dönmez"
    git config --global user.email "doenmezb@phys.ethz.ch"
    ```

2. **Default Editor:**

    Set VSCode as the default editor for Git:

    ```bash
    git config --global core.editor "code --wait"
    ```

3. **Verify Configuration:**

    Review your Git settings to ensure they're correctly applied:

    ```bash
    git config --list
    ```

## Secure Remote Access with SSH

By setting up SSH on your Arch Linux system, you can securely access and manage your
computer remotely.

1. **Prepare SSH Configuration Directory:**

    Initialize the `.ssh` directory with appropriate permissions:

    ```bash
    mkdir -p ~/.ssh
    chmod 700 ~/.ssh
    ```

2. **Enable SSH Service:**

    Enable and start the SSH daemon to ensure it's ready for remote connections:

    ```bash
    systemctl enable --now sshd.service
    ```

## Clipboard Indicator and History

1. **Clone the Gnome Clipboard Indicator AUR Package:**

    Use `git` to clone the Gnome clipboard indicator package repository into a
    dedicated AUR directory:

    ```bash
    cd ~/.aur
    git clone https://aur.archlinux.org/gnome-shell-extension-clipboard-indicator.git
    ```

2. **Build and Install Clipboard Indicator:**

    Navigate to the cloned repository, compile, and install the package:

    ```bash
    cd ~/.aur/gnome-shell-extension-clipboard-indicator
    makepkg -sirc
    git clean -dfx
    ```

    After installation, we clean the build directory with `git clean` to maintain a
    tidy workspace.

3. **Restart Computer:**

    Reboot the computer to apply the changes.

4. **Enable Clipboard Indicator Extension:**

    Enable the extension containing the `"clipboard-indicator"` string:

    ```bash
    gnome-extensions enable $(gnome-extensions list | grep -m 1 clipboard-indicator)
    ```

5. **Customizing Preferences:**

    In **Settings > Limits:**

    - Set **History Size** to `100`.

    In **Settings > Shortcuts:**

    - Set **Toggle the menu** shortcut to <kbd>Super</kbd> + <kbd>V</kbd>.

## Set Up 1Password

1Password is a password manager that helps you securely store and manage passwords.

### Installation

1Password is accessible through the [Arch User Repository (AUR)](
https://aur.archlinux.org/packages/1password). For more details, refer to the
[1Password for Linux Download Page](
https://support.1password.com/install-linux/#arch-linux).

1. **Import 1Password Signing Key**:

    Begin by importing the official 1Password signing key to ensure the authenticity of
    the package:

    ```bash
    curl -sS https://downloads.1password.com/linux/keys/1password.asc | gpg --import
    ```

    The source is signed with the GPG key 3FEF9748469ADBE15DA7CA80AC2D62742012EA22.

2. **Clone the 1Password AUR Package:**

    Use `git` to clone the 1Password package repository into a dedicated AUR directory:

    ```bash
    cd ~/.aur
    git clone https://aur.archlinux.org/1password.git
    ```

3. **Build and Install 1Password:**

    Navigate to the cloned repository, compile, and install the package:

    ```bash
    cd ~/.aur/1password
    makepkg -sirc
    git clean -dfx
    ```

    After installation, we clean the build directory with `git clean` to maintain a
    tidy workspace.

### Configuration

1. **Starting 1Password**:

    Launch 1Password either from the application menu or via the terminal with the
    `1password` command.

2. **Initial Setup**:

    Follow the on-screen instructions to set up your 1Password account and start using
    the password manager:

    - Log in to your account.

    - Enable **Unlock using system authentication**.

3. **Customizing Preferences:**

    In **Settings > General**:

    - Set **Click the icon to:** to **Show the main window**.

    - Set the **Autofill** keyboard shortcut to <kbd>Ctrl</kbd> + <kbd>.</kbd>.

### SSH Agent Settings

1. **Set Up SSH Agent:**

    In **Settings > Developer**:

    - Click on **Set Up SSH Agent...**.

    - Select **Use Key Names** to allow 1Password to save your SSH key names to disk.

    - Select **Edit Automatically** to allow 1Password to update your `~/.ssh/config`
    file.

2. **Verify SSH Configuration:**

    Check the contents of the `~/.ssh/config` file and correct any errors:

    ```bash
    vim ~/.ssh/config
    ```

3. **Set Up Git Commit Signing:**

    In 1Password, find your GitHub/GitLab key and open it:

    - Click on **Configure...** in the **Next Step: Sign Your Git Commits** box.

    - Select **Edit Automatically** to allow 1Password to update your `~/.gitconfig`
    file.

4. **Verify Git Configuration:**

    Check the contents of the `~/.gitconfig file and correct any errors:

    ```bash
    vim ~/.gitconfig
    ```

5. **Add SSH Key to GitHub/GitLab:**

    Add your GitHub/GitLab SSH key to the webpage:

    - Navigate to the SSH keys settings page and click on the **Add SSH key** button.

    - Paste the SSH key from 1Password into the Key field.

    - Repeat this process for both **Authentication Keys** and **Signing Keys**.

### Systemd 1Password Service

1. To configure 1Password to launch at startup using systemd, enter the following
command:

    ```bash
    echo -e "\
    [Unit]\n\
    Description=1Password Service\n\
    \n\
    [Service]\n\
    Type=simple\n\
    ExecStart=/usr/bin/1password --silent \n\
    ExecStop=/usr/bin/killall 1password \n\
    Restart=on-failure\n\
    RestartSec=5\n\
    \n\
    [Install]\n\
    WantedBy=default.target\n\
    " | sudo tee /etc/systemd/user/1password.service > /dev/null
    ```

2. Review the contents of the `/etc/systemd/user/1password.service` file and
correct any errors:

    ```bash
    sudo vim /etc/systemd/user/1password.service
    ```

3. Enable the newly created service to initiate at startup:

    ```bash
    systemctl --user enable 1password.service
    ```

## Mozilla Firefox

Mozilla Firefox is a widely-used open-source web browser developed by Mozilla
Corporation.

### Installation

Ensure you have the latest version of Firefox:

```bash
sudo pacman -Syu firefox
```

During the installation process, if prompted to select a provider for ttf-font,
choose `noto-fonts`.

### Configuration

1. **Set Up Sync:**

    - Open Firefox.

    - Click on the menu button (three horizontal lines) in the top-right corner.

    - Select **Sync and save data: Sign In**.

    - Log in with your Firefox Account credentials or create a new account to s
    ynchronize bookmarks, passwords, and other settings across devices.

2. **Enable Bookmarks Toolbar:**

    - Right-click on the menu button (three horizontal lines) in the top-right corner.

    - Select **Bookmarks Toolbar > Always Show**.

    - Right-click on **Getting Started** bookmark and select **Delete Bookmark**.

3. **Customize Toolbar:**

    - Right-click on the menu button (three horizontal lines) in the top-right corner.

    - Select **Customize Toolbar...**.

    - Drag the **Home** item to the left of address bar.

    - Right-click on **Import bookmarks**, **Flexible Space**, **Save to Pocket** and
    **Account** items and select **Remove from Toolbar**.

    - Click **Done** to save the changes.

4. **Pin Extensions to Toolbar:**

    - Click on the extensions menu button in the top-right corner.

    - Right-click on **1Password** and select **Pin to Toolbar**.

    - Right-click on any other extensions and unpin them by deselecting
    **Pin to Toolbar**.

5. **Run Extensions in Private Windows:**

    - Click on the extensions menu button in the top-right corner.

    - Select **Manage Extensions**.

    - For **1Password**, **I don't care about cookies** and **uBlock Origin**, allow
    **Run in Private Windows**.

6. **Activate and Configure Extensions:**

    - Ensure that **Grammarly** and **Control Panel for Twitter** extensions are
    enabled and configured.

## Install Visual Studio Code (VSCode)

Visual Studio Code is a popular code editor developed by Microsoft.

1. **Clone the VSCode AUR Package:**

    Use `git` to clone the VSCode package repository into a dedicated AUR
    directory:

    ```bash
    cd ~/.aur
    git clone https://aur.archlinux.org/visual-studio-code-bin.git
    ```

2. **Build and Install VSCode:**

    Navigate to the cloned repository, compile, and install the package:

    ```bash
    cd ~/.aur/visual-studio-code-bin
    makepkg -sirc
    git clean -dfx
    ```

    After installation, we clean the build directory with `git clean` to maintain a
    tidy workspace.

## Set up Google Drive

To manage Google Drive files on Arch Linux, we use Rclone.

### Rclone Installation and Configuration

1. Install Rclone by running the following command:

    ```bash
    sudo pacman -Syu rclone
    ```

2. Configure Rclone by running the following command:

    ```bash
    rclone config
    ```

    Follow the on-screen prompts to set up Rclone for use with your Google Drive
    account.

    - The remote name is: `google-drive`

    - Type of storage is: `drive`

    - For **Google Application Client Id**, log into the [Google API Console](
    https://console.developers.google.com) with your Google account.

    - Option scope is: `drive`

    For more detailed instructions on configuring Rclone for Google Drive, visit the
    [Rclone website](https://rclone.org/drive/).

### Systemd Google Drive Service and Auto Mount

1. Install `fuse` if it's not already present on your system with the following command:

    ```bash
    sudo pacman -Syu fuse3
    ```

2. Configure your Google Drive to mount at startup with systemd by running the
subsequent command:

    ```bash
    echo -e "\
    [Unit]\n\
    Description=Google Drive (Rclone) Service\n\
    \n\
    [Service]\n\
    Type=simple\n\
    ExecStartPre=/usr/bin/mkdir -p %h/\"Google Drive\" \n\
    ExecStart=/usr/bin/rclone mount \
    google-drive: %h/\"Google Drive\" \
    --config=%h/.config/rclone/rclone.conf \
    --vfs-cache-mode full \n\
    ExecStop=/bin/fusermount -uz %h/\"Google Drive\"\n\
    ExecStopPost=/usr/bin/rm -rf %h/\"Google Drive\" \n\
    Restart=on-failure\n\
    RestartSec=5\n\
    \n\
    [Install]\n\
    WantedBy=default.target\n\
    " | sudo tee /etc/systemd/user/google-drive.service > /dev/null
    ```

3. Review the contents of the `/etc/systemd/user/google-drive.service` file and correct
any errors:

    ```bash
    sudo vim /etc/systemd/user/google-drive.service
    ```

4. Enable the newly created service to initiate at startup:

    ```bash
    systemctl --user enable --now google-drive.service
    ```

    Your Google Drive files will now be mounted at `~/Google Drive`.

## Checking System Clock Synchronization

It's crucial to ensure that the system clock is accurately synchronized to avoid issues
with time-based applications and services.

1. To set your time zone:

    ```bash
    timedatectl set-timezone Europe/Zurich
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

## Configuring Vim

1. Configure Vim to prevent loading defaults if `~/.vimrc` is missing:

    ```bash
    sudo sed -i -e \
    's/"let skip_defaults_vim=1/let skip_defaults_vim=1\n/' \
    /etc/vimrc
    ```

2. Configure Vim to use the `xfce4-clipman-plugin` clipboard manager:

    ```bash
    echo -e "\
    \" Configure Vim to use the \`xfce4-clipman-plugin\` clipboard manager\n\
    set clipboard=unnamed,unnamedplus\n\
    " | sudo tee -a /etc/vimrc > /dev/null
    ```

3. Add custom Vim shortcuts for copy and paste operations with <kbd>Ctrl</kbd> +
<kbd>C</kbd>, <kbd>Ctrl</kbd> + <kbd>V</kbd> and <kbd>Ctrl</kbd> + <kbd>X</kbd>:

    ```bash
    echo -e "\
    \" Custom shortcuts for copy and paste operations\n\
    vmap <C-c> \"+yi\n\
    vmap <C-x> \"+c\n\
    vmap <C-v> c<ESC>\"+p\n\
    imap <C-v> <C-r><C-o>+\n\
    " | sudo tee -a /etc/vimrc > /dev/null
    ```

4. Turn on color syntax highlighting in Vim:

    ```bash
    echo -e "\
    \" Turn on color syntax highlighting\n\
    syntax on\n\
    " | sudo tee -a /etc/vimrc > /dev/null
    ```

5. Run the following command to check the contents of the `/etc/vimrc` file and correct
any errors:

    ```bash
    sudo vim /etc/vimrc
    ```

## Network Configuration

### How to Set the Hostname on Your System

To set the hostname on your system, please follow these steps:

1. Create the `/etc/hostname` file and add your desired hostname to it. For
instance, if you want to set the hostname as `bahadir-desktop`, then run the
following command:

    ```bash
    echo "bahadir-desktop" | sudo tee /etc/hostname > /dev/null
    ```

2. Verify that the `/etc/hostname` file contains the correct hostname by running
the following command:

    ```bash
    sudo vim /etc/hostname
    ```

    Ensure that the file contains only the desired hostname as shown below:

    ```properties
    bahadir-desktop
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
    127.0.1.1       bahadir-desktop\n\n\
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
    127.0.1.1       bahadir-desktop

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
    sudo vim /etc/pacman.conf
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
    sudo vim /etc/pacman.conf
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

### Clean Pacman Cache

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

## Bluetooth Configuration

In Arch Linux, the `bluez` package provides the Bluetooth protocol stack and tools for
managing Bluetooth devices. To install and configure Bluetooth, follow these steps:

### Installing Packages for Bluetooth Protocol

1. Install the `bluez` package:

    ```bash
    sudo pacman -Syu bluez
    ```

2. Enable and start `bluetooth.service`:

    ```bash
    systemctl enable --now bluetooth.service
    ```

## Table of Contents

### [1. Home](./README.md)

### [2. Installation Guide](./INSTALLATION.md)

### [3. First Steps](./FIRSTSTEPS.md)

### [4. Graphical User Interface](./GUI.md)

### [5. Optional Tools and Configurations](./OPTIONAL.md)

### [6. Additional Information](./APPENDIX.md)

### [7. License](./LICENSE)
