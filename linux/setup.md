# Install

## Install Repos

https://rpmfusion.org/Configuration

https://code.visualstudio.com/docs/setup/linux

## Install Packages

```bash
sudo dnf install fedora-workstation-repositories

sudo dnf upgrade
sudo dnf remove kwrite
sudo dnf install adobe-source-code-pro-fonts adobe-source-han-sans-cn-fonts adobe-source-sans-pro-fonts adobe-source-serif-pro-fonts calibre chromium code dnf-automatic editorconfig emacs fd-find fish flameshot git goldendict ibus-qt ibus-rime im-chooser meld java-latest-openjdk-devel libreoffice lpf-spotify-client mpv pandoc podman steam telegram-desktop toolbox wireshark
lpf update
```

## Install JetBrains Toolbox

# Configure

## Various

```
System Settings - Input Devices - Keyboard - Advanced - Caps Lock is also a Ctrl
                - Worksapce Theme - Look And Feel - Breeze
                - Display and Monitor - Scale Display
                - Applications - choose Chromium
                - Remove Kontact, Konsole Favorites & Sotware Updates

https://wiki.archlinux.org/index.php/KRunner
Krunner Meta key: kwriteconfig5 --file kwinrc --group ModifierOnlyShortcuts --key Meta "org.kde.krunner,/App,,display"
Plasma Krunner - System Settings Module - Enable Spell Checker
                                        - Add Bing Dict Web Shortcut

chsh -s `which fish`

ibus-setup - Input Method - Add rime
           - General - Unclick Embed preedit text in application window
                     - Next input method - <Shift>

Emacs - Remove s-s,s-p in KDE

Telegram - Settings - Chat Settings - Choose emoji set - Android
Bitwarden - File - Settings - Clear Clipboard - 1 minute
                            - Enable Tray Icon
Calibre - Preferences - Behavior - Preferred output format - AZW3
https://github.com/lupoDharkael/flameshot#on-kde-plasma-desktop

systemctl enable --user --now emacs

sudo systemctl enable --now dnf-automatic-install.timer
```

## Font

```
bash
mkdir -p ${XDG_DATA_HOME:-~/.local/share}/fonts
cp PingFang.ttc ${XDG_DATA_HOME:-~/.local/share}/fonts
cp HYShuSongErS.ttf ${XDG_DATA_HOME:-~/.local/share}/fonts
fc-cache -v
ctrl-d
```

## KDEConnect

```
sudo firewall-cmd --zone=public --permanent --add-port=1714-1764/tcp
sudo firewall-cmd --zone=public --permanent --add-port=1714-1764/udp
sudo systemctl restart firewalld.service
```

# Misc

## Virtualbox

```
sudo dnf config-manager --add-repo=https://download.virtualbox.org/virtualbox/rpm/fedora/virtualbox.repo
sudo dnf install VirtualBox-6.0 kernel-devel
Settings - Use Bridged Adapter mode
         - Add shared directory
         - File - Preferences - Input - disable Auto Capture Keyboard
```

## Set Caps Lock as CTRL for FreeRDP

```
https://github.com/FreeRDP/FreeRDP/issues/505#issuecomment-148165369
sudo dnf install evtest
Event: time 1555395780.189580, type 4 (EV_MSC), code 4 (MSC_SCAN), value 3a
Event: time 1555395780.189580, type 1 (EV_KEY), code 58 (KEY_CAPSLOCK), value 1
Event: time 1555395780.189580, -------------- SYN_REPORT ------------
Event: time 1555395780.343887, type 4 (EV_MSC), code 4 (MSC_SCAN), value 3a
Event: time 1555395780.343887, type 1 (EV_KEY), code 58 (KEY_CAPSLOCK), value 0

Event: time 1555395844.412410, type 4 (EV_MSC), code 4 (MSC_SCAN), value 1d
Event: time 1555395844.412410, type 1 (EV_KEY), code 29 (KEY_LEFTCTRL), value 1
Event: time 1555395844.412410, -------------- SYN_REPORT ------------
Event: time 1555395844.508691, type 4 (EV_MSC), code 4 (MSC_SCAN), value 1d
Event: time 1555395844.508691, type 1 (EV_KEY), code 29 (KEY_LEFTCTRL), value 0
Event: time 1555395844.508691, -------------- SYN_REPORT ------------

Bus 001 Device 003: ID 0483:5229 STMicroelectronics 84EC-S
Event: time 1558399312.911660, type 4 (EV_MSC), code 4 (MSC_SCAN), value 70039
Event: time 1558399312.911660, type 1 (EV_KEY), code 58 (KEY_CAPSLOCK), value 1
Event: time 1558399312.911660, -------------- SYN_REPORT ------------
Event: time 1558399313.056548, type 4 (EV_MSC), code 4 (MSC_SCAN), value 70039
Event: time 1558399313.056548, type 1 (EV_KEY), code 58 (KEY_CAPSLOCK), value 0
Event: time 1558399313.056548, -------------- SYN_REPORT ------------
Event: time 1558399325.974548, type 4 (EV_MSC), code 4 (MSC_SCAN), value 700e0
Event: time 1558399325.974548, type 1 (EV_KEY), code 29 (KEY_LEFTCTRL), value 1
Event: time 1558399325.974548, -------------- SYN_REPORT ------------
Event: time 1558399326.109563, type 4 (EV_MSC), code 4 (MSC_SCAN), value 700e0
Event: time 1558399326.109563, type 1 (EV_KEY), code 29 (KEY_LEFTCTRL), value 0
Event: time 1558399326.109563, -------------- SYN_REPORT ------------

sudo emacs /etc/udev/hwdb.d/90-caps-to-control.hwdb
evdev:atkbd:dmi:*
  KEYBOARD_KEY_3a=leftctrl

evdev:input:*v0483p5229*
  KEYBOARD_KEY_70039=leftctrl
```

## Fedora Server Settings

```
sudo systemctl disable --now cockpit
```
