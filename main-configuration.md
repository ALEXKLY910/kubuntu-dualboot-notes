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

1. Kubuntu has APT package manager, here's a bunch of commands:
   '''
   Arch:
   pacman -Syu update system
   pacman -S package install package
   pacman -R package remove package
   pacman -Ss name search package

   Kubuntu/Ubuntu:
   sudo apt update refresh package lists
   sudo apt upgrade install available updates
   sudo apt install package install package
   sudo apt remove package remove package
   apt search name search package
   '''

   First, run:
   `sudo apt update`
   `sudo apt full-upgrade`

2. Then install the drivers:
   `sudo ubuntu-drivers install`
   Reboot.

3. If on the lock screen after reboot/cold boot the wireless keyboard is not connected, maybe wait a little. If it didn't connect even then, do this:
   1. Open the config: `sudo nano /etc/bluetooth/main.conf`
   2. Uncomment this: `AutoEnable=true`
   3. Restart Bluetooth service: `sudo systemctl restart bluetooth`
   4. Make sure the keyboard is trusted: run `bluetoothctl`. Inside run `devices`. Then `trust XX:XX:XX:XX:XX:XX`, where the argument is the MAC address. Then `quit`
   5. Reboot.

4. Download Chrome:
   '''
   wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
   sudo apt install ./google-chrome-stable_current_amd64.deb
   '''
5. Run `swapon --show`. In my case, the swapfile was only 512M, which is a symbolic amount of RAM. We'll configure zram and keep this swapfile as a simple backup.
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

6. Run `sudo ufw status verbose` - it prints the status of the firewall. If it prints "Status: inactive", we'll activate it:
   '''
   sudo ufw default deny incoming
   sudo ufw default allow outgoing
   sudo ufw enable
   '''

7. Install basic tools: git, package building tools, curl and wget for download/requests, archive tools:
   `sudo apt install git build-essential curl wget unzip zip 7zip unrar`
   In my case only "build-essential" and "unrar" weren't already installed

8. Install LibreOffice: `sudo apt install libreoffice`
9. Install VLC Player: `sudo apt install vlc`
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
