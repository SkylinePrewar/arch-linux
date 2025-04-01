# Configuring the Graphical User Interface

## GRUB Scaling

1. **Install the DejaVu Font Family**

    The DejaVu font family is based on the Bitstream Vera Fonts and provides a broad
    range of characters.

    ```bash
    sudo pacman -Syu ttf-dejavu
    ```

2. **Convert the Font for GRUB**

    GRUB requires fonts in a specific format (`.pf2`) to be used during the boot
    process. To adjust the GRUB font size, you need to convert the `DejaVuSansMono.ttf`
    font to the `.pf2` format:

    ```bash
    sudo grub-mkfont --output=/boot/grub/fonts/dejavu_32.pf2 --size=32 /usr/share/fonts/TTF/DejaVuSansMono.ttf
    ```

3. **Modify the GRUB Configuration File**

    To apply the new font to GRUB, modify the `/etc/default/grub` configuration file:

    ```bash
    sudo sed -i -e '/^GRUB_FONT=/d' \
    -e '/^#\?GRUB_THEME=/a\GRUB_FONT="/boot/grub/fonts/dejavu_32.pf2"' \
    /etc/default/grub
    ```

4. **Verify the Configuration**

    Check the contents of the `/etc/default/grub` file and correct any errors:

    ```bash
    sudo vim /etc/default/grub
    ```

    The `GRUB_FONT` line in the file should look like this:

    ```properties
    # Uncomment one of them for the gfx desired, a image background or a gfxtheme
    #GRUB_BACKGROUND="/path/to/wallpaper"
    #GRUB_THEME="/path/to/gfxtheme"
    GRUB_FONT="/boot/grub/fonts/dejavu_32.pf2"
    ```

5. **Regenerate the GRUB Configuration**

    Regenerate the GRUB configuration file to apply the changes:

    ```bash
    sudo grub-mkconfig -o /boot/grub/grub.cfg
    ```

6. **Reboot Your System**

    Finally, to see the changes in effect, reboot your system.

## Configuring GNOME Settings

1. **Disable Hot Corner**

    Disable the hot corner functionality:

    ```bash
    gsettings set org.gnome.desktop.interface enable-hot-corners false
    ```

2. **Enable Workspaces on All Displays**

    Extend workspaces across all connected displays:

    ```bash
    gsettings set org.gnome.mutter workspaces-only-on-primary false
    ```

## Configure Dash

1. **Remove Unnecessary Icons**:

    Right-click on unwanted icons in the dash and select **Unpin** to remove them.

2. **Pin Essential Applications**:

    Right-click on the following applications and select **Pin to Dash**:

    - **Firefox**

    - **Files**

    - **Console**

    - **VS Code**

    After pinning, you can drag and drop the icons to reorder them according to the
    order above.

## Set User Profile Picture

1. **Copy the Image**

    Copy the image to the `/var/lib/AccountsService/icons/` directory:

    ```bash
    cp -r /home/bahadir/"Google Drive"/Personal/"Profile Pictures"/bahadir_profile_1_1.png /home/bahadir/Downloads
    sudo mv /home/bahadir/Downloads/bahadir_profile_1_1.png /var/lib/AccountsService/icons/bahadir
    ```

2. **Update the User Account File**

    - Edit the user account file `/var/lib/AccountsService/users/bahadir` to point to the
    new image:

        ```bash
        echo -e "\
        [User]\n\
        Icon=/var/lib/AccountsService/icons/bahadir\n\
        " | sudo tee /var/lib/AccountsService/users/bahadir > /dev/null
        ```

    - Review the contents of the `/var/lib/AccountsService/users/bahadir`
    file and correct any errors:

        ```bash
        sudo vim /var/lib/AccountsService/users/bahadir
        ```

3. **Restart the Accounts Daemon**

    Restart the `AccountsService` daemon for changes to take effect:

    ```bash
    sudo systemctl restart accounts-daemon.service
    ```

## Single and Dual Monitor Backgrounds

You can customize your desktop backgrounds in GNOME, whether you are using a single
monitor or a dual-monitor setup.

1. **Organize Background Directories**

    First, create directories to store your single-monitor and dual-monitor backgrounds,
    then copy the images to these directories:

    ```bash
    sudo rm -r /usr/share/backgrounds/single-monitor /usr/share/backgrounds/dual-monitor
    cp -r ~/"Google Drive"/Resources/Backgrounds/16x9 /home/bahadir/Downloads
    cp -r ~/"Google Drive"/Resources/Backgrounds/32x9 /home/bahadir/Downloads
    sudo sh -c 'mkdir -p /usr/share/backgrounds/single-monitor && cp /home/bahadir/Downloads/16x9/* $_'
    sudo sh -c 'mkdir -p /usr/share/backgrounds/dual-monitor && cp /home/bahadir/Downloads/32x9/* $_'
    rm -r /home/bahadir/Downloads/16x9 /home/bahadir/Downloads/32x9
    ```

2. **Create a Shell Script to Change the Background**

    - Create a shell script that automatically selects and applies a random background
    based on your monitor setup:

        ```bash
        echo -e \
        '#!/bin/bash

        # Define the directories where the backgrounds are stored
        single_monitor_backgrounds_dir="/usr/share/backgrounds/single-monitor"
        dual_monitor_backgrounds_dir="/usr/share/backgrounds/dual-monitor"

        # Choose and set the background for dual monitor setup
        if [[ -d "$dual_monitor_backgrounds_dir" ]]; then
            background=$(find "$dual_monitor_backgrounds_dir" -type f | shuf -n 1)
            if [[ -n "$background" && -f "$background" ]]; then
                gsettings set org.gnome.desktop.background picture-uri "file://$background"
                gsettings set org.gnome.desktop.background picture-options 'spanned'
            else
                echo "No suitable background file found in $dual_monitor_backgrounds_dir."
            fi
        else
            echo "Directory $dual_monitor_backgrounds_dir does not exist."
        fi
        ' | sudo tee /usr/local/bin/random_gnome_background.sh > /dev/null
        ```

    - Review the contents of the `/usr/local/bin/random_gnome_background.sh`
    file and correct any errors:

        ```bash
        sudo vim /usr/local/bin/random_gnome_background.sh
        ```

3. **Make the Script Executable**

    Run `chmod +x` on the script to make it executable:

    ```bash
    sudo chmod +x /usr/local/bin/random_gnome_background.sh
    ```

4. **Configure the Script to Run at Startup and Every Hour**

    - To configure the script to launch at startup and to run every hour using systemd,
    create a systemd service and timer:

        ```bash
        echo -e "\
        [Unit]\n\
        Description=Service to Randomize GNOME Background\n\
        \n\
        [Service]\n\
        Type=oneshot\n\
        ExecStart=/usr/local/bin/random_gnome_background.sh\n\
        " | sudo tee /etc/systemd/user/random-gnome-background.service > /dev/null
        ```

    - Then create a timer that triggers the service every hour:

        ```bash
        echo -e "\
        [Unit]\n\
        Description=Hourly Randomization of GNOME Background\n\
        \n\
        [Timer]\n\
        OnBootSec=1min\n\
        OnUnitActiveSec=1h\n\
        Unit=random-gnome-background.service\n\
        \n\
        [Install]\n\
        WantedBy=timers.target\n\
        " | sudo tee /etc/systemd/user/random-gnome-background.timer > /dev/null
        ```

5. **Review the Files**

    - Review the contents of the `/etc/systemd/user/random-gnome-background.service`
    file and correct any errors:

        ```bash
        sudo vim /etc/systemd/user/random-gnome-background.service
        ```

    - Review the contents of the `/etc/systemd/user/random-gnome-background.timer`
    file and correct any errors:

        ```bash
        sudo vim /etc/systemd/user/random-gnome-background.timer
        ```

6. **Enable the Timer**

    Enable and start the timer so that the script runs automatically:

    ```bash
    systemctl --user enable --now random-gnome-background.timer
    ```

## Table of Contents

### [1. Home](./README.md)

### [2. Installation Guide](./INSTALLATION.md)

### [3. First Steps](./FIRSTSTEPS.md)

### [4. Graphical User Interface](./GUI.md)

### [5. Optional Tools and Configurations](./OPTIONAL.md)

### [6. Additional Information](./APPENDIX.md)

### [7. License](./LICENSE)
