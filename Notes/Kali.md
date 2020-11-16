[TOC]

# Kali Linux

Debian is a Unix-like operating system composed entirely of free software packaged by a team of volunteers.



# Necessary Packages

- curl
- htop
- nmap
- gobuster
- searchsploit (exploitdb)
- git
- tmux

# Important File Locations

## Config Files

* ssh: `/etc/ssh/sshd_config`
* vim : `etc/vim/vimrc/`
* .bashrc (Global config) : `/etc/bash.bashrc/`
  * For customizing PS1 (bash shell prompt) for all users
* .bashrc (User config) : `~/.bashrc/`

# How To

## Processes

### Detailed information about a process

```bash
ps -Flww -p PID
```

* **-F** : extra full output format
* **-l** : long output format
* **-ww** : Wide output

#### Virtual filesystem located at `/proc/PID/`

Reference: https://ops.tips/blog/what-is-slash-proc/#what-is-procfs



## Filesystem

### SymLink (symbolic link)

A symbolic link, also known as a symlink or soft link, is a special kind of file (entry) that points to eh actual file or directory on a disk (like a shortcut on Windows).

#### Example of SymLink

```bash
lrwxrwxrwx 1 root root          7 Nov 17 22:36 python -> python2
```

#### Create SymLink

The `ln` command is a standard Linux utility for  making link between files.

``` bash
ln -s /path/to/file /path/to/symlink
```

- *-s* or *--symbolic* : Make symbolic links instead of hard links 
- *SOURCE* : A filepath the SymLink will point to
- *LINK_NAME* : A string of the SymLink's name
  - LINK_NAME must be unique

### Custom bash shell prompt (PS1)

```bash
vim /etc/bash.bashrc

# Set a fancy prompt (overwrite the one in /etc/profile)
# but only if not SUDOing and have SUDO_PS1 set; then assume smart user.
# https://stackoverflow.com/questions/4842424/list-of-ansi-color-escape-sequences
# Color codes : https://i.stack.imgur.com/KTSQa.png
bold="\[\e[1m\]"
user_color="\[\e[38;5;33m\]"
host_color="\[\e[38;5;254m\]"
pwd_color="\[\e[38;5;210m\]"
reset="\[\e[0m\]"


if ! [ -n "${SUDO_USER}" -a -n "${SUDO_PS1}" ]; then
#  PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
    PS1="$bold$user_color\u"      # root
    PS1+="$reset@"                # @
    PS1+="$bold$host_color\h"     # hostname
    PS1+="$reset:"                # :
    PS1+="$pwd_color\w"           # ~/path/to/directory
    PS1+="$reset\$ "              # $
fi

export PS1
unset bold user_color host_color pwd_color reset
```

## Launch GUI to local desktop from remote server

Running a Kali server from the cloud with minimal resources includes removing the desktop environment, which means programs with GUIs (Burp Suite, Firefox, gedit, etc.) cannot be opened and viewed. We can download a [display server](https://en.wikipedia.org/wiki/Category:X_servers) in order to launch these programs locally through SSH and X11 Forwarding.

References: 
- [X11 forwarding to view GUI application running on server hosts](https://fabianlee.org/2018/10/14/ubuntu-x11-forwarding-to-view-gui-applications-running-on-server-hosts/)
- [SSH PuTTY User Manual: Using X11 Forwarding in SSH](https://www.ssh.com/ssh/putty/putty-manuals/0.68/Chapter3.html#using-x-forwarding)
- [Using `ssh -X` to launch remote GUI on local desktop](https://askubuntu.com/questions/886313/what-is-the-simplest-way-to-have-remote-gui-access-to-ubuntu-16-04-server-from)

### X11 Protocol

The **“X server”** is what is run on the graphic desktop environment. This is your local Ubuntu desktop host, Windows, or Mac. The **“X client”** is the Linux host that is console-based and has no graphical interface of its own.  

From the local Windows desktop (X server) we are going to SSH into the Linux server (X client), making sure that X11 forwarding setting is enabled.

### Enable X11 Forwarding on Remote (Linux - X Client)

Open the file `etc/ssh/sshd_config`and uncomment the following lines

```bash
X11Forwarding yes
X11DisplayOffset 10
```

Restart the SSH service and verify it's running

```bash
# Restart the SSH service with either of the following commands
service ssh restart
systemctl restart ssh
# Verify the SSH service is running
service ssh status
systemctl status ssh
```

### Enable X11 Forwarding on Local (Windows - X Server)

#### Xming and XLaunch

1. Install `XLaunch` with default configuration
   1. Multiple windows
   2. Display number = 0
2. Launch `Xming`
   1. Verify `Xming` is running by locating its icon (X-shaped) in the system tray

#### PuTTY

1. Open PuTTY configuration and navigate to X11 settings :`Connection --> SSH --> X11`
2. Click `Enable X11 forwarding`
3. Set `X display location` to `localhost:0.0`

### Verify X11 Forwarding is enabled

#### DISPLAY Environment Variable (Linux - X Client)

Ensure the `$DISPLAY` variable has a  value of `localhost:X.0`

```bash
root@beanbag:~# echo $DISPLAY
localhost:10.0
```



## SOCKS5 Proxy

Why? Access HTB websites and perform Burp Suite interceptions from local computer instead of going through VNC and remote desktop.

From local computer, start an `ssh` connection with the following arguments. Connect to SOCKS5 proxy with FoxyProxy on Firefox.

```bash
ssh -p 50922 -D 8080 -C -N root@beanbag
```

- **-p** specifies the remote port to establish `ssh` connection
- **-D** open a SOCKS proxy on local port 8080
- **-C** compress data in the tunnel
- **-N** tells `ssh` to remain idle so that we don't execute a remote command

## tmux

Terminal multiplexer, standard utility.

Config located at `~/.tmux.conf`

### Create Session

`tmux new <cr>|-s NAME`

### List Sessions

`tmux ls|list-sessions`

### Attach to Session

`tmux attach|attach-session -t NUM|SESSION_NAME`

### Basic Commands

```bash
# TODO File bug in typora with CTRL+H replacing when
# trying to replace `ctrl+b` with `PREFIXKEY`

# Prefix key
CTRL+b
# Man pages of tmux commands
PREFIXKEY ?
# Create new window
PREFIXKEY c
# Rename window
PREFIXKEY ,
# Next window
PREFIXKEY n
# Previous window
PREFIXKEY p
# Change to window number
PREFIXKEY NUM
# Detach from tmux session
PREFIXKEY d

```



### `vi` search function

`PREFIXKEY [`

enter edit mode

`space`

copy mode, enter to save to buffer

`PREFIXKEY ]`

paste



### Panes

```bash
# Vertical split
PREFIXKEY %
# Horizontal split
PREFIXKEY "
# Navigate panes
PREFIXKEY ARROWKEYS
# Resize panes
(HOLD) PREFIXKEY (TAP) ARROWKEYS
# Zoom in/out
PREFIXKEY Z
# Move left 
P {
# Move right
P }
# Cycle layout
P SPACE

```



## VNC Server

Reference: [DigitalOcean: How to Install and Configure VNC](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-vnc-on-ubuntu-18-04)

### Installing the Desktop Environment and VNC Server

See Reference above...

### Configuring the VNC Server

Configuration files located at `~/.vnc` and `~/.vnc/xstartup`

### Connecting to the VNC Desktop Securely

Reference: [VNC Setup and Connecting](http://docs.planetimager.org/gpilib/vncs.html)

#### Start VNC Server

```bash
root@beanbag:~$ vncserver

Warning: beanbag:1 is taken because of /tmp/.X1-lock
Remove this file if there is no X server beanbag:1

New 'X' desktop is beanbag:2

Starting applications specified in /root/.vnc/xstartup
Log file is /root/.vnc/beanbag:2.log
```

#### Kill VNC Server

```bash
root@beanbag:~$ vncserver -kill :2
Killing Xtightvnc process ID 4900
```

#### Create SSH connection

Create SSH connection on local computer that securely forwards to the `localhost` connection for VNC.

```bash
ssh -p 50922 -L 5901:localhost:5901 -C -N -l root beanbag
```

- **-p** specifies the remote port to establish `ssh `connection
- **-L** specifies the port bindings, bind port `590X` of the remote connection to port `5901` on your local machine, where X is the screen number of the VNC server
- **-C** compress data in the tunnel
- -N tells `ssh` to remain idle so that we don't execute a remote command
- **-l** specifies the remote login name

#### Connect to VNC Server (Windows, TightVNC Viewer)

Launch `TightVNC Viewer` and connect to `localhost::5901`

# References

* Getting Debian: https://www.debian.org/distrib/
* Installation location: https://cdimage.debian.org/debian-cd/current/amd64/iso-cd
  * File: https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-9.6.0-amd64-xfce-CD-1.iso

* Installation guide: https://www.debian.org/releases/stable/amd64/

