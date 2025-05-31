# Brightness-Battery-AC-power-script
Script to automatically ajust screenbrightness on battery or AC power

#!/bin/bash

LAST_STATE=""
LOW_BATTERY_TRIGGERED=0

notify() {
    notify-send "Batterijstatus" "$1" -u normal -h int:transient:1
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
            notify "Op batterijvermogen – helderheid op 75%"
        else
            brightnessctl set 100%
            notify "Op netstroom – helderheid op 100%"
            LOW_BATTERY_TRIGGERED=0
        fi
        LAST_STATE="$NEW_STATE"
    fi

    if [ "$NEW_STATE" = "BATTERY" ]; then
        if [ "$BATTERY_PERCENT" -lt 20 ] && [ "$LOW_BATTERY_TRIGGERED" -eq 0 ]; then
            brightnessctl set 30%
            notify "Batterij onder 20% – helderheid verlaagd naar 30%"
            LOW_BATTERY_TRIGGERED=1
        elif [ "$BATTERY_PERCENT" -gt 25 ] && [ "$LOW_BATTERY_TRIGGERED" -eq 1 ]; then
            brightnessctl set 75%
            notify "Batterij hersteld – helderheid terug naar 75%"
            LOW_BATTERY_TRIGGERED=0
        fi
    fi

    sleep 5
done
