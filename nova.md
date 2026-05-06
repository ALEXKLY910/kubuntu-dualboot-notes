# TDR Nova on Kubuntu with Carla, yabridge and qpwgraph

We are going to run the Windows VST3 version of TDR Nova through yabridge, host it in Carla, and later route system audio through it with PipeWire/qpwgraph.

## 1. Install required native packages

```bash
sudo apt update
sudo apt install carla carla-vst qpwgraph pipewire-jack pavucontrol libnotify-bin wget tar unzip
```

Check:

```bash
carla --version
qpwgraph --version
```

## 2. Install yabridge

```bash
mkdir -p ~/.local/share
wget -O ~/Downloads/yabridge-5.1.1.tar.gz https://github.com/robbert-vdh/yabridge/releases/download/5.1.1/yabridge-5.1.1.tar.gz
tar -C ~/.local/share -xavf ~/Downloads/yabridge-5.1.1.tar.gz
echo 'export PATH="$PATH:$HOME/.local/share/yabridge"' >> ~/.bashrc
source ~/.bashrc
```

Check:

```bash
yabridgectl --version
```

## 3. Install Wine 9.21 for yabridge/Nova

```bash
mkdir -p ~/.local/share/wine-builds
wget -O ~/Downloads/wine-9.21-staging-tkg-amd64.tar.xz https://github.com/Kron4ek/Wine-Builds/releases/download/9.21/wine-9.21-staging-tkg-amd64.tar.xz
tar -C ~/.local/share/wine-builds -xavf ~/Downloads/wine-9.21-staging-tkg-amd64.tar.xz
```

## 4. Create the TDR Nova Wine prefix

```bash
WINEPREFIX="$HOME/.wine-tdr" WINEARCH=win64 "$HOME/.local/share/wine-builds/wine-9.21-staging-tkg-amd64/bin/wineboot" -u
WINEPREFIX="$HOME/.wine-tdr" "$HOME/.local/share/wine-builds/wine-9.21-staging-tkg-amd64/bin/wineserver" -w
mkdir -p "$HOME/.wine-tdr/drive_c/Program Files/Common Files/VST3"
```

Download the Windows no-installer VST3 version of TDR Nova. Extract it, then copy the .vst3 plugin folder into:
`~/.wine-tdr/drive_c/Program Files/Common Files/VST3/`

## 5. Bridge TDR Nova with yabridge

```bash
yabridgectl add "$HOME/.wine-tdr/drive_c/Program Files/Common Files/VST3"
WINELOADER="$HOME/.local/share/wine-builds/wine-9.21-staging-tkg-amd64/bin/wine" yabridgectl sync
```

Check:
`yabridgectl status`
Expected result should be `TDR Nova.vst3 :: VST3, legacy, 64-bit, synced`

## 6. Set realtime privileges

```bash
sudo groupadd -r realtime
sudo usermod -aG realtime "$USER"
sudo nano /etc/security/limits.d/99-realtime-privileges.conf
```

Paste this:
'''
@realtime - rtprio 98
@realtime - memlock unlimited
@realtime - nice -11
'''

Reboot.

## 7. Configure Carla

```bash
WINELOADER="$HOME/.local/share/wine-builds/wine-9.21-staging-tkg-amd64/bin/wine" pw-jack carla
```

Open its settings and add ~/.vst3 into the plugin paths so that it detects the Nova plugin yabridge exposes. Then set: Settings → Configure Carla → Engine → Process mode → “Continuous Rack”. Just another way to ensure that Carla won't mess with qpwgraph's patchbay (it generally shouldn't because we later configure it to be exclusive, but anyway)
Add TDR Nova plugin inside Carla, open it and configure it (reference `nova/nova_*.jpg`). Then save the project to `~/.config/carla/nova.carxp`.

---

In qpwgraph, make sure the connections you made are present:

NovaFX output → Carla input
Carla output → your current real output device

Enable these options for the patchbay: Activated, Exclusive

If this output is actually your external DAC, save it as:

`~/.config/qpwgraph/nova-dac.qpwgraph`
