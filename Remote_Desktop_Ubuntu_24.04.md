 # Install Remote Desktop support to Ubuntu 24.04 via XRPD

 ## Install dependencies for both GNOME and XFCE4 support

 Adapted from:
 https://medium.com/itversity/how-to-set-up-rdp-on-ubuntu-24-04-for-remote-access-b008411727b7
 https://linuxvox.com/blog/xrdp-ubuntu-install/
 

```bash
sudo apt update
sudo apt install xrdp xserver-xorg-input-all xorgxrdp dbus-x11 xorgxrdp xserver-xorg-core xfce4 xfce4-goodies ubuntu-gnome-desktop gnome-session gnome-terminal gnome-control-center 

sudo adduser xrdp ssl-cert

sudo ufw allow 3389/tcp
sudo ufw reload

sudo systemctl start xrdp
sudo systemctl enable xrdp

```

 ## Modify system-wide configuration

```bash
sudo nano /etc/gdm3/custom.conf
```
Uncomment the line `WaylandEnable=false`
Save and restart: `sudo systemctl restart gdm3`


Allow xrpd to clean-up lock files:

```bash
sudo nano /etc/sudoers.d/xrdp-cleanup
```

Add the following:
```bash
xrdp ALL=(ALL) NOPASSWD: /usr/bin/rm -rf /run/xrdp/sockdir/*, /usr/bin/rm -rf /tmp/.X*-lock, /usr/bin/rm -rf /tmp/.X11-unix/X*
```

```bash
sudo chmod 0440 /etc/sudoers.d/xrdp-cleanup
```


```bash
sudo vim /etc/profile
```

Add the following to the end:

```bash
# XRDP Configuration
# Use one of the following in each user's ~/.profile
#export REMOTE_GUI="gnome"
#export REMOTE_GUI="xfce4"

if [ ${REMOTE_GUI} == "xfce4" ]; then
        # Force software rendering
        export LIBGL_ALWAYS_SOFTWARE=1

        # UNSET THESE to fix the blank screen/black mouse bug
        unset DBUS_SESSION_BUS_ADDRESS
        unset XDG_RUNTIME_DIR
elif [ ${REMOTE_GUI} == "gnome" ]; then
        # Force software rendering (essential for GNOME over RDP)
        export LIBGL_ALWAYS_SOFTWARE=1

        # Set GNOME specific variables
        export GNOME_SHELL_SESSION_MODE=ubuntu
        export XDG_CURRENT_DESKTOP=ubuntu:GNOME
        export XDG_SESSION_TYPE=x11
        export XDG_CONFIG_DIRS=/etc/xdg/xdg-ubuntu:/etc/xdg

        # Clear session variables
        unset DBUS_SESSION_BUS_ADDRESS
        unset XDG_RUNTIME_DIR
fi
```

```bash
sudo vim /etc/xrdp/startwm.sh
```

Modify to match the following:

```bash
#!/bin/sh

# Standard profile loading
if [ -r /etc/profile ]; then . /etc/profile; fi
if [ -r ~/.profile ]; then . ~/.profile; fi

# Extract the display number (e.g., from :11.0 to 11)
DISPLAY_NUM=$(echo $DISPLAY | sed 's/://' | cut -d. -f1)

# Clean only the locks for the current session's display
if [ -n "$DISPLAY_NUM" ]; then
    sudo /usr/bin/rm -f /tmp/.X${DISPLAY_NUM}-lock
    sudo /usr/bin/rm -f /tmp/.X11-unix/X${DISPLAY_NUM}
fi

# Modify the user ~/.profile to select RDP environment using:
# export REMOTE_GUI="gnome"
# export REMOTE_GUI="xfce4"

if [ "$REMOTE_GUI" = "xfce4" ]; then
    exec startxfce4
elif [ "$REMOTE_GUI" = "gnome" ]; then
    test -x /etc/X11/Xsession && exec /etc/X11/Xsession
    exec /bin/sh /etc/X11/Xsession
else
    # Fallback to system default if variable isn't set
    exec /etc/X11/Xsession
fi
```


 ## Modify user configuration

```bash
vim ~/.profile
```

Add the following to the end:
```bash
# Select RDP environment for the user
#export REMOTE_GUI="gnome"
export REMOTE_GUI="xfce4"
```


