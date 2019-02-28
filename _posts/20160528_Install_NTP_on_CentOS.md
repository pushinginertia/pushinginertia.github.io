---
layout: post
title: Install NTP on CentOS
---


# Install NTP and its documentation.

```sh
yum install ntp ntp-doc
```

# Configure NTP servers to query from.

Head to http://www.pool.ntp.org/en/ and select the country based on the geographical location of the server. For example, the following servers would be used for a server located in the U.S.:

```
server 0.us.pool.ntp.org
server 1.us.pool.ntp.org
server 2.us.pool.ntp.org
server 3.us.pool.ntp.org
```

Edit `/etc/ntp.conf`, replacing the default servers with the ones identified above.

# Review security configuration.

The default settings are most likely sufficient, but review them against the NTP access restrictions documentation and modify as needed.

# Configure leap seconds.

Download the leap seconds file from http://www.ietf.org/timezones/data/leap-seconds.list and save it to the server in a location like `/var/lib/ntp/leapfile`. Ensure that this file is readable by `ntpd`.

Modify `/etc/ntp.conf` and add the line:

```
leapfile /var/lib/ntp/leapfile
```

See: http://support.ntp.org/bin/view/Support/ConfiguringNTP#Section_6.14.

# Configure logging.

Log output is placed in `/var/log/messages` by default but can be overridden if desired. My preference is to leave it there as it doesn’t really make sense to have a separate log file for a specific service, but the following line in `/etc/ntp.conf` will log elsewhere.

```
logfile /var/log/ntp.log
```

See: https://www.redhat.com/archives/rhl-list/2007-December/msg03702.html

# Start the ntpd service

```sh
service ntpd start
```

# Check synchronization status with peers.

```sh
# ntpq -p
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
+host2.kingrst.c 198.60.73.8      2 u   38   64  377   31.327   -3.206  12.424
*lithium.constan 18.26.4.105      2 u   66   64  373   10.723  -11.870   3.080
-propjet.latt.ne 199.233.236.226  3 u    4   64  377  112.837    3.736  30.346
+F-Current.sjela 204.123.2.72     2 u    5   64  377   98.932    1.000  14.950
```

# Sniff UDP port 123.

Watch for NTP packets on UDP 123. It should take a minute or so to see some traffic to the NTP servers.

```sh
# tcpdump dst port 123
listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
12:39:33.371065 IP localhost.ntp > F-Current.sjelab.net.ntp: NTPv4, Client, length 48
12:39:33.441244 IP F-Current.sjelab.net.ntp > localhost.ntp: NTPv4, Server, length 48
12:39:35.369876 IP localhost.ntp > propjet.latt.net.ntp: NTPv4, Client, length 48
12:39:35.516572 IP propjet.latt.net.ntp > localhost.ntp: NTPv4, Server, length 48
12:39:37.369878 IP localhost.ntp > lithium.constant.com.ntp: NTPv4, Client, length 48
12:39:37.389644 IP lithium.constant.com.ntp > localhost.ntp: NTPv4, Server, length 48
12:39:47.377980 IP localhost.ntp > host2.kingrst.com.ntp: NTPv4, Client, length 48
12:39:47.475338 IP host2.kingrst.com.ntp > localhost.ntp: NTPv4, Server, length 48
```

# Query current status of ntpd.

```sh
# ntpdc -c sysinfo
system peer:          lithium.constant.com
system peer mode:     client
leap indicator:       11
stratum:              3
precision:            -20
root distance:        0.05229 s
root dispersion:      0.09605 s
reference ID:         [108.61.56.35]
reference time:       daf4104b.6a0f384e  Sat, May 28 2016 12:40:43.414
system flags:         auth monitor ntp kernel stats
jitter:               0.010727 s
stability:            0.000 ppm
broadcastdelay:       0.000000 s
authdelay:            0.000000 s
```
