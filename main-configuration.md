# Prepare Windows.

1. Disable Fast Startup / hybernation:

   > `powercfg.exe /hibernate off`

2. Check the Bitlocker. It should be disabled.

   > `manage-bde -status`

3. Backup important stuff (done)

4. Shrink the C: Windows partition. Open up disk management tool: Win+X -> Disk Management, right click the partition -> shrink volume -> enter 256000 MB (for 250GB).

# Try Kubuntu

1. Download the Kubuntu ISO from https://kubuntu.org/download/, download Rufus from https://rufus.ie/, write the ISO on a USB stick with it, partition scheme - GPT, Target system - UEFI.

2. Boot from the USB (it should say UEFI in the name of the device), choose "Try or Install Kubuntu". First try it. Check that the display is working, the internet, sound and devices are operational.

Then proceed to installation.

# Install Kubuntu

1. On the live desktop, launch "Install Kubuntu". Choose minimal instalation. On the Partitions tab, choose Manual partitioning.

2. Select the dedicated 250GB free space, click "Create", choose the size of 512 MiB, file system FAT32, mount point "/boot/efi", "boot" flag.

3. Then select the remaining 249.5GB free space, click "Create", choose the mount point to be "/".

4. Click "Install". After it finishes, restart.

# Configure GRUB

1. Go into UEFI menu, into Boot settings, UEFI Hard Disk Drive BBS Priorities. Make it boot into Kubuntu first, basically.

2. Then boot into Kubuntu, edit this file: "/etc/default/grub".
   You should have these essential settings:
   '''
   GRUB_TIMEOUT_STYLE=menu
   GRUB_TIMEOUT=60
   GRUB_DISABLE_OS_PROBER=false
   '''
   After that, run `sudo update-grub`. Reboot.

### GRUB works, but is a bit ugly. Can be later replaced with **rEFInd**.

# Configure Kubuntu

1.  Kubuntu has APT package manager, here's a bunch of commands:
    '''
    Arch:
    pacman -Syu - update system
    pacman -S package - install package
    pacman -R package - remove package
    pacman -Ss name - search package

    Kubuntu/Ubuntu:
    sudo apt update - refresh package lists
    sudo apt upgrade - install available updates
    sudo apt install package - install package
    sudo apt remove package - remove package
    apt search name - search package
    '''

    First, run:
    `sudo apt update`
    `sudo apt full-upgrade`

2.  Then install the drivers:
    `sudo ubuntu-drivers install`
    Reboot.

3.  If on the lock screen after reboot/cold boot the wireless keyboard is not connected, maybe wait a little. If it didn't connect even then, do this:
    1. Open the config: `sudo nano /etc/bluetooth/main.conf`
    2. Uncomment this: `AutoEnable=true`
    3. Restart Bluetooth service: `sudo systemctl restart bluetooth`
    4. Make sure the keyboard is trusted: run `bluetoothctl`. Inside run `devices`. Then `trust XX:XX:XX:XX:XX:XX`, where the argument is the MAC address. Then `quit`
    5. Reboot.

4.  Download Chrome:
    '''
    wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
    sudo apt install ./google-chrome-stable_current_amd64.deb
    '''
5.  Run `swapon --show`. In my case, the swapfile was only 512M, which is a symbolic amount of RAM. We'll configure zram and keep this swapfile as a simple backup.
    1. Install the zram package: `sudo apt install systemd-zram-generator`
    2. Create the config: `sudo nano /etc/systemd/zram-generator.conf`. Paste this:
       '''
       [zram0]
       zram-size = min(ram / 2, 8192)
       compression-algorithm = zstd
       swap-priority = 100
       '''
       Then reboot.
    3. Run `swapon --show` to verify.

6.  Run `sudo ufw status verbose` - it prints the status of the firewall. If it prints "Status: inactive", we'll activate it:
    '''
    sudo ufw default deny incoming
    sudo ufw default allow outgoing
    sudo ufw enable
    '''

7.  Install basic tools: git, package building tools, curl and wget for download/requests, archive tools:
    `sudo apt install git build-essential curl wget unzip zip 7zip unrar`
    In my case only "build-essential" and "unrar" weren't already installed

8.  Install LibreOffice: `sudo apt install libreoffice`
9.  Install VLC Player: `sudo apt install vlc`
10. Install Microsoft's VS Code:
    '''
    cd ~/Downloads
    wget https://code.visualstudio.com/sha/download?build=stable\&os=linux-deb-x64 -O code-stable-amd64.deb
    sudo apt install ./code-stable-amd64.deb
    '''
11. Install Steam: `sudo apt install steam-installer`. Steam might warn you that you don't have the required 32-bit drivers for the GPU. Click OK. Then run `dpkg -l | grep libnvidia-gl`, it might show you that the drivers were installed. Launch `steam`.

12. Install Happ. Go to `https://github.com/Happ-proxy/happ-desktop/releases`, download a .deb package and install it with this command: `sudo apt install ./happ.deb`

13. Install Discord:
    '''
    cd ~/Downloads
    wget "https://discord.com/api/download?platform=linux&format=deb" -O discord.deb
    sudo apt install ./discord.deb
    '''
14. Install fonts:
    `sudo apt install fonts-noto-color-emoji fonts-liberation fonts-crosextra-carlito fonts-crosextra-caladea fonts-jetbrains-mono fonts-powerline`

    Also, install JetBrains Mono Nerd:
    '''
    cd ~/Downloads

    wget https://github.com/ryanoasis/nerd-fonts/releases/latest/download/JetBrainsMono.tar.xz -O JetBrainsMono.tar.xz

    mkdir -p ~/.local/share/fonts/JetBrainsMonoNerd
    tar -xf JetBrainsMono.tar.xz -C ~/.local/share/fonts/JetBrainsMonoNerd
    '''
    fc-cache -r

15. Install Ghostty: `sudo apt install ghostty`. In Default Applications (System Settings) make it be the default terminal emulator.

16. Go to System Settings:
    1. Mouse & Touchpad: disable the mouse acceleration, tweak the pointer speed
    2. Keyboard: enable layouts, add Russian layout, bind Alt+Shift for changing the layout, disable the popup.
    3. In Display & Monitor disable actions for when the cursor touches the edges or corners.
    4. In accessibility remove shake pointer to find it feature
    5. In Disks & Cameras turn automount all known devices on login and on attach.
    6. In animations, configure animations.
    7. In Power Management, configure sleep, dim, turn off screen and power profile.

17. Install Fuzzel: `sudo apt install fuzzel`. Bind it by going into Shortcuts, Add New, Program or Script, use `fuzzel` as the script.

18. Configure the shortcuts in Keyboard -> Shortcuts in System Settings.

19. Install CopyQ: `sudo apt install copyq`. Also, install wl-clipboard: `sudo apt install wl-clipboard`.
    Configure CopyQ: add the shortcut, disable saving the items after reboot.

20. Remap the Caps key to be a modifier and add the shortcuts for Home and End:

    > `sudo apt install keyd`
    > `sudo nano /etc/keyd/default.conf`

        '''
         [ids]
         *

         [main]
         capslock = layer(caps)

         [caps]
         left = home
         right = end
         C-left = C-home
         C-right = C-end
        '''

    > `sudo systemctl enable --now keyd`
    > `sudo systemctl restart keyd`

21. Create a theme toggle script:
    `mkdir -p ~/.local/bin`
    `nano ~/.local/bin/toggle-theme`
    Reference scripts/toggle-theme.
    `chmod +x ~/.local/bin/toggle-theme`
    Bind it.

22. Configure multi-desktop workflow:
    1. First, create 10 virtual desktops: System Settings → search “Virtual Desktops”
    2. Search: Close Window; Set: Meta + C
    3. Search: Fullscreen; Set: Meta + F
    4. Search: Switch to Desktop 1; Set: Meta + 1
    5. Search: Window to Desktop 1; Set: Meta + Shift + 1
    6. To view all desktops, press Meta + G
    7. Search: Maximize Window; Set: Meta + M
    8. Download Compact Pager in Plasma Widgets store
    9. Search: Keep Window Above; Set: Ctrl+Meta+T

23. Configure Fuzzel to have a dark theme.
    1. Create `~/.config/fuzzel/themes/`.
    2. Create `~/.config/fuzzel/themes/dracula.ini` and paste this inside:

    ```
    [colors]
    background=1f1f1fff
    text=ccccccff
    match=4daafcff
    selection=0078d4cc
    selection-text=ffffffff
    selection-match=4daafcff
    border=2b2b2bff
    prompt=ccccccff
    placeholder=989898ff
    input=ccccccff
    counter=9d9d9dff
    ```

    3. Create `~/.config/fuzzel/themes/light.ini` and leave it empty.
    4. Create `~/.config/fuzzel/fuzzel.ini` and paste this inside:

    ```
    [main]
    include=~/.config/fuzzel/themes/current.ini
    ```

    5. Create a symlink, say, to dark:

       > `ln -sf ~/.config/fuzzel/themes/dracula.ini ~/.config/fuzzel/themes/current.ini`

    6. Switch the destination of the symlink between `dracula.ini` and `light.ini` based on the current system theme in the `toggle-theme` script. Add this somewhere after declaring `is_new_dark` variable:

    ```
    # Wire up fuzzel's theme

    if [[ "$is_new_dark" == "true" ]]; then
       fuzzel_scheme="~/.config/fuzzel/themes/dracula.ini"
    else
       fuzzel_scheme="~/.config/fuzzel/themes/light.ini"
    fi

    [[ -f "$fuzzel_scheme" ]] || {echo "Missing fuzzel color scheme file: $fuzzel_scheme" >&2; exit 1; }

    ln -sf $fuzzel_scheme $HOME/.config/fuzzel/themes/current.ini
    ```

24. Install **zapret**.

```bash
cd ~/Apps
git clone https://github.com/Sergeydigl3/zapret-discord-youtube-linux.git && cd zapret-discord-youtube-linux
./service.sh download-deps --default
./service.sh
```

```
3. Изменить конфигурацию. enp... - ethernet; wlp... - wi-fi.
   After that a file called conf.env is created.
```

```bash
./service.sh run --config conf.env
```

Or create and alias:

```bash
nano ~/.bashrc
```

Put inside:

```
alias zapret="/home/alex/Apps/zapret-discord-youtube-linux/service.sh run --config /home/alex/Apps/zapret-discord-youtube-linux/conf.env"
```

```bash
source ~/.bashrc
```

25. Configure screenshot shortcuts:
    Bind: `spectacle --background --fullscreen --copy-image --nonotify && notify-send "Screenshot" "Fullscreen copied to clipboard"` to Win+S
    Bind: `bash -lc 'mkdir -p "$HOME/Pictures/Screenshots"; spectacle --background --fullscreen --output "$HOME/Pictures/Screenshots/shot-$(date +%F_%H-%M-%S).png" --nonotify && notify-send "Screenshot" "Fullscreen saved to Pictures/Screenshots"'` to Win+Ctrl+S
    Bind: `spectacle --background --region --copy-image --nonotify && notify-send "Screenshot" "Selection copied to clipboard"` to Win+Shift+S
    Bind: `bash -lc 'mkdir -p "$HOME/Pictures/Screenshots"; spectacle --background --region --output "$HOME/Pictures/Screenshots/area-$(date +%F_%H-%M-%S).png" --nonotify && notify-send "Screenshot" "Selection saved to Pictures/Screenshots"'` to Win+Shift+Ctrl+S

    Also, in spectacle, Accept on click-and-release.

26. Configure OCR.
    1. `sudo apt install tesseract-ocr tesseract-ocr-eng tesseract-ocr-rus tesseract-ocr-jpn kdialog wl-clipboard libnotify-bin`
    2. `nano ~/.local/bin/ocr-region`. Paste the contents of scripts/ocr-region
    3. `chmod +x ~/.local/bin/ocr-region`

27. Configure ghostty:
    1. Check out solarized themes:

       > `ghostty +list-themes --plain`

    2. Create a file at `~/.config/ghostty/themes/default-light.conf`. Paste inside it:

    ```
    theme="Monokai Pro Light Sun"
    ```

    3. Create a file at `~/.config/ghostty/themes/default-dark.conf`. Paste inside it:

    ```
    theme="Dark Modern"
    ```

    4. Create a symlink to one of them:

    > `ln -sf ~/.config/ghostty/themes/default-light.conf ~/.config/ghostty/themes/current.conf` 5. Create a file at `~/.config/ghostty/config` and paste inside it:

    ```
    config-file = themes/current.conf

    # stop some TUI apps from using near-background colors on text
    minimum-contrast = 2

    font-size=16

    working-directory = inherit
    window-inherit-working-directory = false
    tab-inherit-working-directory = true
    split-inherit-working-directory = true
    ```

    6. Paste this to our toggle script beneath the `is_new_dark` variable declaration:

    ```
    #Wire up ghostty's theme

    if [[ "$is_new_dark" == "true" ]]; then
       ghostty_scheme="$HOME/.config/ghostty/themes/default-dark.conf"
    else
       ghostty_scheme="$HOME/.config/ghostty/themes/default-light.conf"
    fi

    [[ -f "$ghostty_scheme" ]] || { echo "Missing ghostty color scheme file: $ghostty_scheme" >&2; exit 1; }

    ln -sf $ghostty_scheme $HOME/.config/ghostty/themes/current.conf

    #reload ghostty's config
    pkill -USR2 -x ghostty

    ```

28. Configure fuzzy search across your filesystem:
    1. > `sudo apt install fzf`

    2. Add to `~/.bashrc`

    <!-- to find where fzf installed the scripts, run `dpkg -L fzf | grep -E 'key-bindings|completion'` -->

    ```
    # fzf keybindings + completion
    [ -f /usr/share/doc/fzf/examples/completion.bash ] && source /usr/share/doc/fzf/examples/completion.bash
    [ -f /usr/share/doc/fzf/examples/key-bindings.bash ] && source /usr/share/doc/fzf/examples/key-bindings.bash
    ```

    Then reload:

    > `source ~/.bashrc`

    Now you can type `Ctrl+T` in a shell and fuzzy search!

29. Configure Syncthing.

> `sudo apt install syncthing`

1.  If you want to enable the service run this:
    > `systemctl --user enable --now syncthing.service`
2.  If you want to toggle the service by a keybind and make it so it won't autostart, reference and tie the script: `arch_linux_configs/toggle-syncthing.sh`

Create it at `~/.local/bin/toggle-syncthing`. Make it executable. Tie it to SUPER+SHIFT+Y

You use the service in a browser by typing this URL:

`http://127.0.0.1:8384`

Turn on versioning (staggered).

You can modify .stignore to include stuff you don't want to sync.

If you hit a limit of files uploaded at once, modify `/etc/sysctl.d/99-inotify.conf` by adding `fs.inotify.max_user_watches=524288`. And running `sud sysctl --system`.

30. Nova. Reference nova.md

31. Lightning
    Try OpenRGB:

```bash
sudo apt update
sudo apt install openrgb
```

JRAINBOW1: 40
JRAINBOW2: 10

32. Shared folder (FS) for Syncthing.

Boot into Windows, shrink the main partition:
Right-click start button -> Disk Management
Shrink -> 102400 MB
Click on unallocated space, "New Simple Volume"

```
S:
NTFS
Label: Shared
Size: ~100 GB
Quick format: enabled
Compression: disabled
```

Create a permanent mount folder in Kubuntu and test-mount it.

```bash
sudo mkdir -p /mnt/shared
```

Install NTFS support if it's not already installed:

```bash
sudo apt install ntfs-3g
```

Mount the partition manually:

```bash
sudo mount -t ntfs-3g UUID=B4E8D716E8D6D5A8 /mnt/shared -o uid=1000,gid=1000,umask=022,windows_names
```

P.S. in order to find out its UUID, run `lsblk -o NAME,SIZE,FSTYPE,LABEL,UUID,MOUNTPOINTS`
Check that it's mounted:

```bash
df -h | grep shared
```

Now we make the mount permanent.
First back up `/etc/fstab`:

```bash
sudo cp /etc/fstab /etc/fstab.backup
```

Open it:

```bash
sudo nano /etc/fstab
```

Add this line at the bottom:

```
UUID=B4E8D716E8D6D5A8 /mnt/shared ntfs-3g uid=1000,gid=1000,dmask=022,fmask=133,windows_names,nofail,x-systemd.device-timeout=10 0 0
```

```bash
systemctl daemon-reload
```

Test it:

```bash
sudo umount /mnt/shared
sudo mount -a
df -h | grep shared
ls -la /mnt/shared
```

And create a symlink in Kubuntu:

```bash
ln -s /mnt/shared/Sync ~/Sync
```
