# :battery: :computer: Brightness-Battery-AC-power-script with notifications

Script to automatically adjust screen brightness based on battery or AC power status on a Linux system with Hyprland and notifies you.\
The script checks every 5 seconds on the power state.

## 📜 Script path: `~/.config/hypr/scripts/battery-brightness-watcher.sh`

```bash
#!/bin/bash

LAST_STATE=""
LOW_BATTERY_TRIGGERED=0

notify() {
    notify-send -u normal -a "Battery status" "$1" "$2" -h int:transient:1
}

while true; do
    AC_STATE=$(cat /sys/class/power_supply/ACAD/online)
    BATTERY_PERCENT=$(cat /sys/class/power_supply/BAT1/capacity)
    
    if [ "$AC_STATE" = "1" ]; then
        NEW_STATE="AC"
    else
        NEW_STATE="BATTERY"
    fi

    if [ "$NEW_STATE" != "$LAST_STATE" ]; then
        if [ "$NEW_STATE" = "BATTERY" ]; then
            brightnessctl set 75%
            notify "󱊣 On battery" "brightness at 75%"
        else
            brightnessctl set 100%
            notify "󱐥 On AC power" "brightness at 100%"
            LOW_BATTERY_TRIGGERED=0
        fi
        LAST_STATE="$NEW_STATE"
    fi

    if [ "$NEW_STATE" = "BATTERY" ]; then
        if [ "$BATTERY_PERCENT" -lt 20 ] && [ "$LOW_BATTERY_TRIGGERED" -eq 0 ]; then
            brightnessctl set 30%
            notify "󱊡 Battery below 20%" "brightness lowerd at 30%"
            LOW_BATTERY_TRIGGERED=1
        elif [ "$BATTERY_PERCENT" -gt 25 ] && [ "$LOW_BATTERY_TRIGGERED" -eq 1 ]; then
            brightnessctl set 75%
            notify "Battery recovered" "brightness back at 75%"
            LOW_BATTERY_TRIGGERED=0
        fi
    fi

    sleep 5
done
```

## ⚙️ Systemd user service: `~/.config/systemd/user/battery-brightness.service`

```ini
[Unit]
Description=Battery Brightness Watcher
After=graphical-session.target
Requires=graphical-session.target

[Service]
Type=simple
ExecStart=/home/kristof/.config/hypr/scripts/battery-brightness-watcher.sh
Restart=always
RestartSec=5

# Make sure session bus will be found:
Environment=DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/%U/bus
# (Optional: XDG_RUNTIME_DIR mostly /run/user/%U)
#Environment=XDG_RUNTIME_DIR=/run/user/%U

# Call Wayland variables only when necessary; DISPLAY=:0 is not needed in Wayland.
#Environment=DISPLAY=:0
#Environment=WAYLAND_DISPLAY=wayland-0

# (Make sure the script itself is executed with the right permissions; 
#  User is already in group video for Brightness.)
KillMode=process

[Install]
WantedBy=default.target


```

Replace `YOUR_USERNAME` with your actual Linux username.

## ▶️ Enable the systemd user service

```bash
systemctl --user daemon-reexec
systemctl --user daemon-reload
systemctl --user enable --now battery-brightness.service
systemctl --user status battery-brightness.service
```

Make sure to add yourself to video and input groups with 
```
sudo usermod -aG video,input $User
```
