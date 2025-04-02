# Configuring the Graphical User Interface

## GRUB Scaling
  > [!TIP]
   > Maybe I will skip this part because my hyprland setup script already takes care of it.

1. **Install the DejaVu Font Family**

    The DejaVu font family is based on the Bitstream Vera Fonts and provides a broad
    range of characters.

    ```bash
    # sudo pacman -Syu ttf-dejavu
    ```

2. **Convert the Font for GRUB**

    GRUB requires fonts in a specific format (`.pf2`) to be used during the boot
    process. To adjust the GRUB font size, you need to convert the `DejaVuSansMono.ttf`
    font to the `.pf2` format:

    ```bash
    # sudo grub-mkfont --output=/boot/grub/fonts/dejavu_32.pf2 --size=32 /usr/share/fonts/TTF/DejaVuSansMono.ttf
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
    # sudo micro /etc/default/grub
    ```

    The `GRUB_FONT` line in the file should look like this:

    ```properties
    # Uncomment one of them for the gfx desired, a image background or a gfxtheme
    #GRUB_BACKGROUND="/path/to/wallpaper"
    #GRUB_THEME="/path/to/gfxtheme"
    GRUB_FONT="/boot/grub/fonts/dejavu_32.pf2"
    ```

    

6. **Regenerate the GRUB Configuration**

    Regenerate the GRUB configuration file to apply the changes:

    ```bash
   # sudo grub-mkconfig -o /boot/grub/grub.cfg
    ```

7. **Reboot Your System**

    Finally, to see the changes in effect, reboot your system.

## Set User Profile Picture

 Copy the image to the `/usr/share/sddm/faces/` directory:
   First, download the image and after
    ```bash
    # sudo mv ~/Downloads/username.face.icon /usr/share/sddm/faces/
    ```

**How to Make a Script Executable?**

  If I need, I'll check this info.
  
  Run `chmod +x` on the script to make it executable:

```bash
# sudo chmod +x /path/to/script.sh
``` 

## Table of Contents

### [1. Home](./README.md)

### [2. Installation Guide](./INSTALLATION.md)

### [3. First Steps](./FIRSTSTEPS.md)

### [4. Graphical User Interface](./GUI.md)

### [5. Optional Tools and Configurations](./OPTIONAL.md)

### [6. Additional Information](./APPENDIX.md)

### [7. License](./LICENSE)
