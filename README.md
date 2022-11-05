# SmokePing-Nginx-RockyLinux
How to install SmokePing on Rocky Linux 8 with Nginx

# Smokeping with Nginx on Rocky Linux 8
The following installation was done under **Rocky Linux version 8.6** (Green Obsidian).

**SmokePing source**: https://oss.oetiker.ch/smokeping/

## Installation Assumptions

**Installation directory**: /opt/smokeping

**Domain**: smokeping.yourdomain.com

**Account under which the web server runs**: nginx

## Install Dependencies and Requirements

dnf -y install epel-release

dnf -y groupinstall "Development tools"

dnf -y install perl httpd httpd-devel mod\_fcgid rrdtool perl-CGI-SpeedyCGI fping rrdtool-perl perl-Sys-Syslog fcgiwrap openssl-devel

dnf -y install perl-CPAN perl-local-lib perl-Time-HiRes

## Download Smokeping Source

1. export VER="2.8.2"
2. mkdir -p /opt/smokeping.build
3. cd /opt/smokeping.build
4. wget [https://oss.oetiker.ch/smokeping/pub/smokeping-$VER.tar.gz](https://oss.oetiker.ch/smokeping/pub/smokeping-%24VER.tar.gz)
5. tar xzvf smokeping-$VER.tar.gz

## Build Smokeping 

1. mkdir -p /opt/smokeping
2. cd /opt/smokeping.build/smokeping-$VER
3. ./configure --prefix=/opt/smokeping
4. gmake install
5. adduser -r -d /var/lib/smokeping -s /usr/sbin/nologin -c "SmokePing daemon" smokeping
6. rm -f /opt/smokeping.build

## Create Diretcories

1. mkdir -p /opt/smokeping/etc/config.d
2. mkdir /var/lib/smokeping
3. chown smokeping:smokeping /var/lib/smokeping
4. mkdir -p /var/cache/smokeping/images
5. mkdir -p /var/lib/smokeping/\_\_cgi
6. chown -R smokeping:nginx /var/lib/smokeping/\_\_cgi
7. chmod u+rwx,g+rws,o+xr-w /var/lib/smokeping/\_\_cgi
8. chown nginx:nginx /var/cache/smokeping/images

## Copy Files

1. cp -a /opt/smokeping/etc/basepage.html.dist /opt/smokeping/etc/basepage.html
2. cp -a /opt/smokeping/etc/smokemail.dist /opt/smokeping/etc/smokemail
3. cp -a /opt/smokeping/etc/smokeping\_secrets.dist /opt/smokeping/etc/smokeping\_secrets
4. chmod 640 /opt/smokeping/etc/smokeping\_secrets
5. cp -a /opt/smokeping/etc/tmail.dist /opt/smokeping/etc/tmail
6. cp -a /opt/smokeping/htdocs/smokeping.fcgi.dist /opt/smokeping/htdocs/smokeping.fcgi
7. ln -s /var/cache/smokeping/images /opt/smokeping/htdocs/images


## Create Configuration Files

#### File: /opt/smokeping/etc/config

```
@include /opt/smokeping/etc/config.d/General
@include /opt/smokeping/etc/config.d/Alerts
@include /opt/smokeping/etc/config.d/Database
@include /opt/smokeping/etc/config.d/Presentation
@include /opt/smokeping/etc/config.d/Probes
@include /opt/smokeping/etc/config.d/Slaves
@include /opt/smokeping/etc/config.d/Targets
```

#### File: /opt/smokeping/etc/config.d/Slaves

```
*** Slaves ***
secrets=/opt/smokeping/etc/smokeping_secrets
#+boomer
#display_name=boomer
#color=0000ff

#+slave2
#display_name=another
#color=00ff00
```

#### File: /opt/smokeping/etc/config.d/Targets

Customize the file to your needs

```
*** Targets ***

probe = FPing

menu = Top
title = Network Latency Grapher
remark = Welcome to the SmokePing

+ Local

menu = Local
title = Local Network
#parents = owner:/Test/James location:/

++ LocalMachine
menu = Local Machine
title = This host
host = localhost
#alerts = someloss

++ StromdatenInfo
menu = Stromdaten Info
title = Stromdaten Info
host = stromdaten.info

++ CloudflareDNS
menu = Cloudflare DNS
title = Cloudflare DNS server
host = 1.1.1.1

++ GoogleDNS
menu = Google DNS
title = Google DNS server
host = 8.8.4.4
```

#### File: /opt/smokeping/etc/config.d/Database

```
*** Database ***

step = 15
pings = 15

# consfn mrhb steps total

AVERAGE 0.5 1 1008
AVERAGE 0.5 12 4320
MIN 0.5 12 4320
MAX 0.5 12 4320
AVERAGE 0.5 144 720
MAX 0.5 144 720
MIN 0.5 144 720
```

#### File: /opt/smokeping/etc/config.d/Probes

```
*** Probes ***

+ FPing

binary = /usr/sbin/fping
```

#### File: /opt/smokeping/etc/config.d/Presentation

```
*** Presentation ***

template = /opt/smokeping/etc/basepage.html
charset = utf-8
htmltitle = yes
graphborders = no

+ charts

menu = Charts
title = The most interesting destinations

++ stddev
sorter = StdDev(entries=>4)
title = Top Standard Deviation
menu = Std Deviation
format = Standard Deviation %f

++ max
sorter = Max(entries=>5)
title = Top Max Roundtrip Time
menu = by Max
format = Max Roundtrip Time %f seconds

++ loss
sorter = Loss(entries=>5)
title = Top Packet Loss
menu = Loss
format = Packets Lost %f

++ median
sorter = Median(entries=>5)
title = Top Median Roundtrip Time
menu = by Median
format = Median RTT %f seconds

+ overview

width = 1000
height = 50
range = 10h

+ detail

width = 1000
height = 200
unison_tolerance = 2

"Last 3 Hours" 3h
"Last 30 Hours" 30h
"Last 10 Days" 10d
"Last 360 Days" 360d

#+ hierarchies
#++ owner
#title = Host Owner
#++ location
#title = Location
```

#### File: /opt/smokeping/etc/config.d/pathnames

```
sendmail = /usr/sbin/sendmail
imgcache = /var/cache/smokeping/images
imgurl = ../images
datadir = /var/lib/smokeping
piddir = /run/smokeping
smokemail = /opt/smokeping/etc/smokemail
tmail = /opt/smokeping/etc/tmail
dyndir = /var/lib/smokeping/__cgi
```

#### File: /opt/smokeping/etc/config.d/General

```
*** General ***

owner = Peter Random
contact = some@address.nowhere
mailhost = my.mail.host
# NOTE: do not put the Image Cache below cgi-bin
# since all files under cgi-bin will be executed ... this is not
# good for images.
cgiurl = http://some.url/smokeping.cgi
# specify this to get syslog logging
syslogfacility = local0
# each probe is now run in its own process
# disable this to revert to the old behaviour
# concurrentprobes = no

@include /opt/smokeping/etc/config.d/pathnames
```

#### File: /opt/smokeping/etc/config.d/Alerts

```
*** Alerts ***
to = alertee@address.somewhere
from = smokealert@company.xy

+someloss
type = loss
# in percent
pattern = >0%,*12*,>0%,*12*,>0%
comment = loss 3 times in a row
```

## Create Systemd Start Script 

#### File: /usr/lib/systemd/system/smokeping.service

```
[Unit]
Description=Latency Logging and Graphing System
Documentation=man:smokeping(1) file:/usr/share/doc/smokeping/examples/systemd/slave_mode.conf
After=network.target

[Service]
# It would in theory be simpler to run smokeping with the --nodaemon option and
# Type=simple, but smokeping does not work properly when in "slave" mode with
# --nodaemon set.
Type=forking
RuntimeDirectory=smokeping
PIDFile=/run/smokeping/smokeping.pid
User=smokeping
Group=smokeping
StandardError=syslog

# If you need to run smokeping in slave/master mode, see the example unit
# override in /usr/share/doc/smokeping/examples/systemd/slave_mode.conf
ExecStart=/opt/smokeping/bin/smokeping --pid-dir=/run/smokeping

ExecReload=/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target
```

## Enable Services

systemctl daemon-reload

systemctl enable fcgiwrap@nginx.socket

## Start Services 

systemctl start fcgiwrap@nginx.socket

systemctl restart smokeping.service# Smokeping Web Installation (Rocky Linux 8)

## Nginx Config

#### File: /etc/nginx/sites-available/smokeping

```
server {
    listen 80;
	include /etc/nginx/default.d/*.conf;
	root /opt/smokeping/htdocs;
	index smokeping.cgi;
	
	server_name smokeping.yourdomain.com;
	error_log /var/log/nginx/smokeping.yourdomain.com.error.log;
	access_log /var/log/nginx/smokeping.yourdomain.com.access.log;
	log_not_found off;

	# optional: allow access only from your client IP
	#allow 89.17.28.65; # your client IP address
	#deny all;

	error_page 404 http://$host/smokeping.cgi;


	# URL: http://smokeping.yourdomain.com/smokeping.cgi
	location ~ \.cgi$ {
		fastcgi_intercept_errors on;
		root /usr/lib;
		include /etc/nginx/fastcgi_params;
		fastcgi_param SCRIPT_FILENAME /opt/smokeping/htdocs/smokeping.fcgi;
		fastcgi_pass unix:/run/fcgiwrap/fcgiwrap-nginx.sock;

	}

	location = / {
		return 301 /smokeping.cgi;
	}
}

```

cd /etc/nginx/sites-enabled

ln -s ../sites-available/smokeping .

nginx -t &amp;&amp; systemctl restart nginx.service
