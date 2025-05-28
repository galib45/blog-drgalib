+++
date = '2025-05-29T00:41:30+06:00'
title = 'Wallpapers in Hyprland'
tags = ["hyprland", "wallpaper", "hyprpaper", "linux"]
[cover]
  image = "/tech/wallpapers_in_hyprland/thumbnail.png"
+++

I will be using {{< extlink title="hyprpaper" url="https://github.com/hyprwm/hyprpaper" >}} for setting wallpaper in hyprland. Install it using your package manager. For me the command is 
```bash
sudo xbps-install hyprpaper
```
Now add the following line to your hyprland config store at `~/.config/hypr/hyprland.conf` to autostart `hyprpaper` when hyprland starts.
```hyprlang
exec-once = hyprpaper
```
Now to set a wallpaper we have to preload the image file to memory first and then set it. Hyprland sets a wallpaper by itself on startup, so we have to unload it first. So, the sequence of commands is unload, preload, set. The command for this is
```bash
hyprctl --instance 0 hyprpaper unload all
hyprctl --instance 0 hyprpaper preload path/to/wallpaper
hyprctl --instance 0 hyprpaper wallpaper , path/to/wallpaper
```
But there is a problem with doing it this way. If you exit Hyprland and then re-enter or logout and then login, this wallpaper will be gone. Because hyprland sets a wallpaper by itself on startup. To make the wallpaper persist, we have to save it to `~/.config/hypr/hyprpaper.conf` file.
```hyprlang
preload = path/to/wallpaper
wallpaper = , path/to/wallpaper
```
So, whenever you set a new wallpaper you have to run the 3 `hyprctl` commands and write 2 lines in `~/.config/hypr/hyprpaper.conf` file. To automate this, we will write a bash script. I saved it to `~/scripts/setwallpaper`.
```bash
#!/bin/bash
set -xe

HYPRPAPER_CONFIG_FILE=~/.config/hypr/hyprpaper.conf
WALLPAPERS_DIR=~/wallpapers
WALLPAPER="$(find $WALLPAPERS_DIR -type f | shuf -n 1)"

echo "preload = $WALLPAPER" > $HYPRPAPER_CONFIG_FILE
echo "wallpaper = , $WALLPAPER" >> $HYPRPAPER_CONFIG_FILE

hyprctl --instance 0 hyprpaper unload all
hyprctl --instance 0 hyprpaper preload $WALLPAPER
hyprctl --instance 0 hyprpaper wallpaper , $WALLPAPER
```
Now make it executable using `chmod +x ~/scripts/setwallpaper`. This randomly selects a wallpaper from the `~/wallpapers` directory, saves it to `hyprpaper` config and sets the wallpaper using `hyprctl` commands. 

Finally, you can configure a keybinding in your hyprland config. I set the `SUPER + SHIFT + W` to run the `~/scripts/setwallpaper`.
```hyprlang
bind = $mainMod SHIFT, W, exec, ~/scripts/setwallpaper
```
