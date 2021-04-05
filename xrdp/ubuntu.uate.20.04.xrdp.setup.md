# Ubuntu Mate 20.04 XRDP setup

Documenting XRDP setup which worked for me on Ubuntu Mate 20.04.

Some parts are taken from: http://c-nergy.be/blog/?p=14093.

## Install xrdp

    sudo apt install xrdp

## Allow console access

    sudo sed -i 's/allowed_users=console/allowed_users=anybody/' /etc/X11/Xwrapper.config

## Create policies

The following policies are needed in order to get rid of Authorization popup which spawns after successfull login.

### Create color manager policy

Create a file: `/etc/polkit-1/localauthority/50-local.d/45-allow.colord.pkla`

Paste the following content:

```
[Allow Colord all Users]
Identity=unix-user:*
Action=org.freedesktop.color-manager.create-device;org.freedesktop.color-manager.create-profile;org.freedesktop.color-manager.delete-device;org.freedesktop.color-manager.delete-profile;org.freedesktop.color-manager.modify-device;org.freedesktop.color-manager.modify-profile
ResultAny=no
ResultInactive=no
ResultActive=yes
```

### Create package manager policy

Create a file: `/etc/polkit-1/localauthority/50-local.d/46-allow-update-repo.pkla`

Paste the following content:

```
[Allow Package Management all Users]
Identity=unix-user:*
Action=org.freedesktop.packagekit.system-sources-refresh
ResultAny=yes
ResultInactive=yes
ResultActive=yes
```

### Create Network manager Wifi Scan policy

Create a file: `/etc/polkit-1/localauthority/50-local.d/47-allow-wifi-scan.pkla`

Paste the following content:

```
[Allow WiFi Scan all Users]
Identity=unix-user:*
Action=org.freedesktop.NetworkManager.wifi.scan
ResultAny=no
ResultInactive=no
ResultActive=yes
```

## Configure Xorg

### Configure the Xorg session

Create a file: `~/.xsession`

Paste the following content:

```
mate-session
```

### Configure the Xorg session remote connection settings

Create a file: `~/.xsessionrc`

Paste the following content:

```
export XDG_SESSION_DESKTOP=mate
export XDG_DATA_DIRS=${XDG_DATA_DIRS}
export XDG_CONFIG_DIRS=/etc/xdg/xdg-mate:/etc/xdg
```

## Fix missing environment variables

Edit the following file: `/etc/pam.d/xrdp-sesman`

And add the following line at the beginning:

```
session required pam_env.so readenv=1 user_readenv=0
```

Example:

```
#%PAM-1.0
session required pam_env.so readenv=1 user_readenv=0
@include common-auth
@include common-account
@include common-session
@include common-password
```

## Fix local .profile not being sourced

Copy `/etc/xrdp/startwm.sh` to `~/startwm.sh` and add the following code:

```
if test -r /home/${USER}/.profile; then
	. /home/${USER}/.profile
fi
```

Example:

```
#!/bin/sh
# xrdp X session start script (c) 2015, 2017 mirabilos
# published under The MirOS Licence

if test -r /etc/profile; then
	. /etc/profile
fi

if test -r /home/${USER}/.profile; then
	. /home/${USER}/.profile
fi

if test -r /etc/default/locale; then
	. /etc/default/locale
	test -z "${LANG+x}" || export LANG
	test -z "${LANGUAGE+x}" || export LANGUAGE
	test -z "${LC_ADDRESS+x}" || export LC_ADDRESS
	test -z "${LC_ALL+x}" || export LC_ALL
	test -z "${LC_COLLATE+x}" || export LC_COLLATE
	test -z "${LC_CTYPE+x}" || export LC_CTYPE
	test -z "${LC_IDENTIFICATION+x}" || export LC_IDENTIFICATION
	test -z "${LC_MEASUREMENT+x}" || export LC_MEASUREMENT
	test -z "${LC_MESSAGES+x}" || export LC_MESSAGES
	test -z "${LC_MONETARY+x}" || export LC_MONETARY
	test -z "${LC_NAME+x}" || export LC_NAME
	test -z "${LC_NUMERIC+x}" || export LC_NUMERIC
	test -z "${LC_PAPER+x}" || export LC_PAPER
	test -z "${LC_TELEPHONE+x}" || export LC_TELEPHONE
	test -z "${LC_TIME+x}" || export LC_TIME
	test -z "${LOCPATH+x}" || export LOCPATH
fi

test -x /etc/X11/Xsession && exec /etc/X11/Xsession
exec /bin/sh /etc/X11/Xsession
```
