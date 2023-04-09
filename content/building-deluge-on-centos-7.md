+++
title = "Building Deluge on CentOS 7"
date = "2016-04-16T16:43:00Z"
description = "I've decided to reinstall the OS on my server from an aging version of Fedora to CentOS. Since I use Deluge for .torrent downloads, and don't want to switch to another client, I've had to build it from source because it isn't included in any of the EL7 repositories."
+++

Since the `deluge` and `deluge-web` packages are omitted from the CentOS/RHEL and EPEL repositories for EL7, and I wouldn't trust some unmaintained 3rd-party repo, I decided to build them from source. The instructions on the [Deluge website](http://dev.deluge-torrent.org/wiki/Installing/Source) fail to mention anything about what's needed for CentOS/RHEL distributions so I guess I'll write up what's needed here.

# Download the source

Get the latest version of the source from the Deluge project's [Source Mirror](http://download.deluge-torrent.org/source/?C=M;O=D). Then extract the source with `tar -xf` .

# Install the dependencies

The dependencies for Deluge and Deluge-Web are stated on the project's website, although only the packages for Debian/Ubuntu are listed. Some of the dependencies are in the EPEL repo, so you'll have to run this command first if you haven't already enabled EPEL.

```
$ sudo yum install epel-release
```

Once the EPEL repo is enabled, run the following command to install the relevant dependencies.

```
$ sudo yum install python python-twisted-core python-twisted-web pyOpenSSL \
  python-setuptools gettext intltool pyxdg python-chardet python-GeoIP \
  rb_libtorrent-python2 python-setproctitle python-pillow python-mako
```

# Build and install Deluge and Deluge-Web

Change into the directory where you extracted the deluge source files, and run the following to build Deluge and Deluge-Web, and then install the application into the usuall locations.

```
$ python setup.py build
$ sudo python setup.py install
```

# Create a service user

You shouldn't be running a service under a standard user account, so create a user for deluge to run as.

```
$ sudo useradd -d /var/lib/deluge -c "deluge daemon account" -r -s /sbin/nologin -m deluge
```

# Install the systemd service files

These are the .service files taken from the Fedora version of Deluge. Create the `deluge-daemon.service` and `deluge-web.service` files inside `/etc/systemd/system`. Once created, you can start and enable both services using the following commands.

```
$ systemctl daemon-reload
$ systemctl enable deluge-daemon-service
$ systemctl enable deluge-web.service
$ systemctl start deluge-daemon.service
$ systemctl start deluge-web.service
```

## deluge-daemon.service

```
[Unit]
Description=Deluge Bittorrent Client Daemon 
After=network.target

[Service]
User=deluge
ExecStart=/usr/bin/deluged -d

[Install]
WantedBy=multi-user.target
```

## deluge-web.service

```
[Unit]
Description=Deluge Bittorrent Client Web Interface
After=network.target

[Service]
User=deluge
ExecStart=/usr/bin/deluge --ui web

[Install]
WantedBy=multi-user.target
```