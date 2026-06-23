# My bluetooth keyboard wasn't connecting after login. Here's the current fix:

```bash
sudo tee /usr/local/sbin/fix-realtek-bluetooth.sh >/dev/null <<'EOF'
#!/bin/sh

set_usb_power_on() {
    for device in /sys/bus/usb/devices/*; do
        [ -f "$device/idVendor" ] || continue
        [ -f "$device/idProduct" ] || continue

        if [ "$(cat "$device/idVendor")" = "0bda" ] && [ "$(cat "$device/idProduct")" = "8771" ]; then
            echo on > "$device/power/control"
        fi
    done
}

set_usb_power_on
sleep 2

if bluetoothctl power on; then
    exit 0
fi

sleep 2
set_usb_power_on
sleep 2

bluetoothctl power on
EOF
```

```bash
sudo chmod +x /usr/local/sbin/fix-realtek-bluetooth.sh
```

```bash
sudo tee /etc/systemd/system/fix-realtek-bluetooth.timer >/dev/null <<'EOF'
[Unit]
Description=Run Realtek Bluetooth fix after boot

[Timer]
OnBootSec=20s
Unit=fix-realtek-bluetooth.service

[Install]
WantedBy=timers.target
EOF
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable fix-realtek-bluetooth.timer
```

```bash
sudo tee /etc/systemd/system/fix-realtek-bluetooth.service >/dev/null <<'EOF'
[Unit]
Description=Fix Realtek Bluetooth dongle power state
After=bluetooth.service multi-user.target
Wants=bluetooth.service

[Service]
Type=oneshot
ExecStartPre=/bin/sleep 3
ExecStart=/usr/local/sbin/fix-realtek-bluetooth.sh

[Install]
WantedBy=multi-user.target
EOF
```

```bash
sudo systemctl daemon-reload
```

```bash
mkdir -p ~/.config/systemd/user

cat > ~/.config/systemd/user/bluetooth-power-on.service <<'EOF'
[Unit]
Description=Power on Bluetooth after user login
After=default.target

[Service]
Type=oneshot
ExecStartPre=/bin/sleep 5
ExecStart=/usr/bin/bluetoothctl power on

[Install]
WantedBy=default.target
EOF
```

```bash
systemctl --user daemon-reload
systemctl --user enable bluetooth-power-on.service
systemctl --user start bluetooth-power-on.service
systemctl --user status bluetooth-power-on.service --no-pager
```

---

basic apt package maintenance:

```bash
sudo apt update
sudo apt upgrade
sudo apt autoremove
```
