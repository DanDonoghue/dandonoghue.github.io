+++
title = "Aria2 Web on EL7"
date = "2016-05-22T04:13:00Z"
description = "How I set up Aria2 with a WebUI on CentOS"
+++

[Aria2](https://aria2.github.io/) is a CLI download manager that can download files over several different protocols. I've been meaning to set something like this up for ages, as it's better to download files on my server than to leave another machine turned on.

# Install Aria2 and Apache's httpd

The following commands enable the EPEL repo, install `aria2` and `httpd`, and adds a firewall rule to allow HTTP traffic from the public zone. If you've set up different zones (like I have), use the zone that you want to allow access from.

```
sudo yum -y install epel-release
sudo yum -y install aria2 httpd
sudo firewall-cmd --zone=public --add-service=http
sudo firewall-cmd --zone=public --add-service=http --permanent
```

# Configure a systemd service for Aria2

## Create a service user

To run Aria2 as a service, you need to give it a user account. This helps security-wise because you can tune what locations it can access using ACLs and such. The following command creates a service user, with the shell `/sbin/nologin`, and the home directory `/path/where/downloads/are/stored`. 

You should change the home directory to wherever you're storing downloaded files.

```
sudo useradd -s /sbin/nologin -r aria2 -d /path/where/downloads/are/stored
```

## Write the systemd `.service` file

Systemd service files are small configuration files that tell systemd how it should run something. The following code block is my service file, which I saved as `/etc/systemd/system/aria2.service`. The important lines are in the `Service` block, specifically the `User` and `WorkingDirectory` lines. You should update `User` to be the user you wish to run Aria2 as, and also update `WorkingDirectory` to point to the location where you want to store downloaded files.

```
[Unit]
Description=Aria2 Download Manager
After=network.target

[Service]
User=aria2
WorkingDirectory=/path/where/downloads/are/stored
ExecStart=/bin/aria2c --enable-rpc --show-console-readout=false

[Install]
WantedBy=multi-user.target
```

## Start Aria2

Whenever you update systemd's `.service` files, you have to run a `daemon-reload`. If you want to have the service start automatically at it's default targets (`WantedBy=multi-user.target`) then you must `enable` it. To simply start the service right now, you use the `start` line. The following code block shows all three `systemctl` commands needed to get Aria2 running and to auto-start it at boot.

```
sudo systemctl daemon-reload
sudo systemctl enable aria2.service
sudo systemctl start aria2.service
```

# Configure Apache's httpd

## WebUI Files

Firstly, you need to make a directory in `/var/www` that will contain the Aria2 WebUI files.

```
sudo mkdir /var/www/aria2
```

Once this directory is made, download the [Aria2 WebUI](https://github.com/ziahamza/webui-aria2) and place them into the newly created folder. Make sure that `index.html` is located at `/var/www/aria2/index.html`.

## Configuration File

Httpd needs to be configured to proxy the `/jsonrpc` endpoint to the port that Aria2 listens on (default: `tcp/6800`). The following virtual-host configuration file covers the basics. Save this file as `/etc/httpd/conf.d/aria2.conf`

```
<VirtualHost *:80>
	DocumentRoot /var/www/aria2
	ProxyPass "/jsonrpc" "http://localhost:6800/jsonrpc"
	ProxyPassReverse "/jsonrpc" "http://localhost:6800/jsonrpc"
</VirtualHost>
```

## Selinux changes

Because the reverse proxy is being used, httpd needs to be able to connect to network ports. This is blocked by default by `selinux`.

```
sudo setsebool -P httpd_can_network_connect on
```

## Start httpd

```
sudo systemctl enable httpd.service
sudo systemctl start httpd.service
```

# Browse to the WebUI

After the previous steps, you should have running Aria2 and httpd services. Opening `http://<ip>/` should show you the WebUI, which should be connected to Aria2 over the reverse proxy to Aria2's jsonrpc URI. If something doesn't work, check the logs of the `httpd` and `aria2` services.

# Checking the logs

## Apache's httpd

Apache stores its logs in various different files instead of the `systemd` approach of using `journald`, so you'll have to check the files the old-school way with `tail`.

```
sudo tail -f /var/log/httpd/error_log
```

## Aria2

Because Aria2 is being run as a systemd service, its `stdout` is being redirected into `journald`. The easiest way to read these logs is with `journalctl`.

```
sudo journalctl -lfu aria2
```

You can also see a status for the service by using `systemctl status <unit>`, like so:

```
sudo systemctl status aria2.service
```
