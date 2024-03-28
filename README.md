## Power Profiles Enhancer

This was a personal fork of https://github.com/marcinx64/pp-to-amd-epp/tree/main and then evolved out of its
scope, thus power-profiles-enhancer was born.

From kernel 6.5+ there's AMD-PSTATE in active mode as default CPU driver on ZEN2+ machines

Power Profiles Daemon cannot talk to AMD-PSTATE driver directly leaving it unmanaged and inconsistent with whole system
energy profile. Moreover, power profiles does not change power limits, governors and DPM for the iGPU.

This project is aiming to close this gap with monitoring Power Profiles current mode in the background and setting:

* Corresponding EPP profile for CPU.
* Corresponding governor for CPU
* Corresponding TDP limits for CPU
* Corresponding DPM level for iGPU


EPP Modes of operation if modify_epp is true:

* PP set to 'power-saver' -> EPP set to 'power' (lowest energy usage preference)
* PP set to 'balanced':
    * if AC is disconnected and use_balance_power=true in config -> EPP set to 'balance_power'
    * if AC is connected or use_balance_power=false in config-> EPP set to 'balance_performance'
* PP set to 'performance' -> EPP set to 'performance'

DPM modes of operation if modify_dpm is true:

* PP set to 'power-saver' and allow_low_dpm=true-> DPM set to low
* PP set to 'balanced' -> DPM set to auto
* PP set to 'performance' and allow_hig_dpm=true-> DPM set to high

Governor modes of operation if use_performance_governor is true:

* PP set to 'power-saver' -> governor set to powersave
* PP set to 'balanced' -> governor set to powersave
* PP set to 'performance' and use_performance_governor=true-> governor set to performance

TDP limits modes of operation:

* PP set to 'power-saver' and modify_powersave_tdp_limit-> TDP set as configured
* PP set to 'balanced' and modify_balanced_tdp_limit-> TDP set as configured
* PP set to 'performance' and modify_performance_tdp_limit-> TDP set as configured

Tested on Arch Linux with 6.6.8 kernel on GA402RJ 6900HS and GA402XY 7940HS

dependencies:
* python3
* power-profiles-daemon
* ryzenadj (optional, only for TDP limits)

## Usefulness

Is this useful? It can be, but you should use it and judge it by yourself.
EPP definitely helps. The others, depends.

## Installation

Run ```bash install.sh``` with root privileges. A new service will be enabled: pp-enhancer.service. By default, every 2 seconds it will check if your system needs changes and applies them. You can configure the duration between checks and more by modifying the /etc/pp-enhancer.ini file. Make sure to restart the pp-enhancer.service after changing your configuration

## Configuration explained

### Acronyms:
* epp: Energy Performance Preference
* pp: Power Profile
* dpm: Dynamic Power Management.
* gov: governor

### Configuration

The following is the commented pp-enhancer.ini and with default values that will be used when the .ini or any option
is missing. During installation, a default configuration is copied to /etc/pp-enhancer.ini. You can modify this as you wish.

[enablers]
* modify_epp=true -> If true, epp will match the selected pp. You can set it to false if EPP is matched with PP via asusctl
                    and you don't want to use the balance_power profile when on battery
* modify_dpm=false  -> If true, dpm will match the selected power-profiles (pp). Allows changing of 
                     power_dpm_force_performance_level. More info: https://dri.freedesktop.org/docs/drm/gpu/amdgpu/thermal.html
* allow_low_dpm_bat=false -> If true, a 'low' DPM value is allowed when in power-save mode and on battery. Not suggested for iGPU less powerful than 780M
* allow_low_dpm_ac=false -> If true, a 'low' DPM value is allowed when in power-save mode and on ac. Not suggested for iGPU less powerful than 780M
* allow_high_dpm_bat=false -> If true, a 'high' DPM value is allowed when in performance mode and on battery. Unsure about how useful this is
* allow_high_dpm_ac=false -> If true, a 'high' DPM value is allowed when in performance mode and on ac. Unsure about how useful this is
* modify_powersave_tdp_limit=true -> If true, tdp limits are changed based on the configured tdp-limits below for the power-save profile
                    If you want to use this option, you must install ryzenadj and make sure you can run ryzenadj --info correctly
* modify_balanced_tdp_limit=false -> If true, tdp limits are changed based on the configured tdp-limits below for the balanced profile
                    If you want to use this option, you must install ryzenadj and make sure you can run ryzenadj --info correctly
* modify_performance_tdp_limit=false -> If true, tdp limits are changed based on the configured tdp-limits below for the performance profile
                    If you want to use this option, you must install ryzenadj and make sure you can run ryzenadj --info correctly
* use_balance_power=true -> If true, the balance_power epp will be used when on battery instead of balance_performance
* use_performance_governor=false -> If true, the performance governor will be used when in performance mode
* ryzen_powersave_on_ac=true -> Will append --power-saving to the TDP limit command when in powersave mode. Requires modify_powersave_tdp_limit=true
* ryzen_performance_on_battery=false -> Will append --max-performance to the TDP limit command when in performance mode. Requires modify_performance_tdp_limit=false


Unit: Watt. For more info 
https://github.com/FlyGoat/RyzenAdj/wiki/Renoir-Tuning-Guide  
Fast Limit >= Slow Limit >= STAPM Limit  
It's suggested not to put a limit too low.  For reference, with a 7940HS you can comfortably use 8W in power-mode on a 
1600p screen. For a 4K screen, I need 10W.  
[tdp-limits] 
* powersave_slow_limit_bat=15 ->  STAPM limit for power-save mode on battery
* powersave_fast_limit_bat=20 -> Slow limit for power-save mode on battery
* powersave_sustained_limit_bat=15 -> Fast limit for power-save mode on battery
* powersave_slow_limit_ac=20 ->  STAPM limit for power-save mode on ac
* powersave_fast_limit_ac=25 -> Slow limit for power-save mode on ac
* powersave_sustained_limit_ac=20 -> Fast limit for power-save mode on ac
* balanced_slow_limit_bat=35 -> STAPM limit for balanced mode on battery
* balanced_fast_limit_bat=40 -> Slow limit for balanced mode on battery
* balanced_sustained_limit_bat=35 -> Fast limit for balanced mode on battery
* balanced_slow_limit_ac=45 -> STAPM limit for balanced mode on ac
* balanced_fast_limit_ac=50 -> Slow limit for balanced mode on ac
* balanced_sustained_limit_ac=45 -> Fast limit for balanced mode on ac
* performance_slow_limit_bat=62 ->  STAPM limit for performance mode on battery
* performance_fast_limit_bat=65 -> Slow limit for performance mode on battery
* performance_sustained_limit_bat=62 -> Fast limit for performance mode on battery
* performance_slow_limit_ac=70 -> STAPM limit for performance mode on
* performance_fast_limit_ac=80 -> Slow limit for performance mode on
* performance_sustained_limit_ac=70 -> -> Fast limit for performance mode on

Unit: seconds.  
[intervals]
* sleep_duration=2 -> The service will sleep for this configured amount of seconds between checks of changes
                    of the power profile


The following is for paths that can change based on the system in use. Simply find out the location of 
power_dpm_force_performance_level on your system and add the DPM config  
[paths]  
#For GA402RJ with 6900HS:
* #DPM=/sys/class/drm/card2/device/power_dpm_force_performance_level  
#For GA402XY 2023 with 7940HS:
* #DPM=/sys/class/drm/card0/device/power_dpm_force_performance_level
