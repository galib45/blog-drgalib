+++
date = '2025-05-29T00:41:30+06:00'
title = 'Wallpapers in Hyprland'
tags = ["hyprland", "wallpaper", "hyprpaper", "linux"]
[cover]
  image = "/tech/wallpapers-in-hyprland/thumbnail.png"
+++
## Installing and autostarting hyprpaper
I will be using {{< extlink title="hyprpaper" url="https://github.com/hyprwm/hyprpaper" >}} for setting wallpaper in hyprland. Install it using your package manager. For me the command is 
```bash
sudo xbps-install hyprpaper
```
Now add the following line to your hyprland config store at `~/.config/hypr/hyprland.conf` to autostart `hyprpaper` when hyprland starts.
```hyprlang
exec-once = hyprpaper
```

## Setting the wallpaper
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

## Custom keybinding to change wallpaper
Finally, you can configure a keybinding in your hyprland config. I set the `SUPER + SHIFT + W` to run the `~/scripts/setwallpaper`.
```hyprlang
bind = $mainMod SHIFT, W, exec, ~/scripts/setwallpaper
```

## Automatically change wallpaper after some time
For this, we need cronjobs that run after a certain period of time. For cronjobs to run, a cron daemon is needed, i use `fcron`. Install it by running
```bash
sudo xbps-install fcron
```
Now enable the fcron service by running
```bash
sudo ln -s /etc/sv/fcron /etc/runit/runsvdir/default/
```
Now add the cronjob by running `fcrontab -e`. This will open a temporary file where all the cronjobs are listed. Initially, this will be empty. From `man 5 fcrontab`, 
```man
ENTRIES BASED ON ELAPSED SYSTEM UP TIME
       Jobs are scheduled to run once every m minutes of fcron's execution
       (which is normally the same as m minutes of system's execution). The
       time a system is suspended (to memory or disk) is considered as down
       time. To configure such a job, use configuration lines of the form:

       @options frequency command

       where frequency is a time value of the form
       value*multiplier+value*multiplier+...+value-in-minutes as "12h02" or
       "3w2d5h1".  The first means "12 hours and 2 minutes of fcron execution"
       while the second means "3 weeks, 2 days, 5 hours and 1 minute of fcron
       execution". The only valid multipliers are: 

VALID TIME MULTIPLIERS
       months (4 weeks): m      
       weeks (7 days): w
       days (24 hours): d      
       hours (60 minutes): h  
       seconds: s

       In place of options, user can put a time value: it will be interpreted
       as @first(<time>). If first option is not set, the value of "frequency"
       is used.
```
We will leave the options empty, so the job will run on frequency. In the frequency, the default unit is minutes. So, without any multiplier if we put `n` there, the job will run every `n` minutes. We will add the following line in the fcrontab.
```cron
@ 10 ~/scripts/setwallpaper
```
Now save the file and exit.

