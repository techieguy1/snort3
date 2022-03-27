# Rerences
- [Basic Snort Rules Syntax and Usage](https://resources.infosecinstitute.com/topic/snort-rules-workshop-part-one/)
- [Official Snort Site](https://www.snort.org/)
- [Snort Cheat Sheet](https://www.comparitech.com/net-admin/snort-cheat-sheet/#tables)
- [Snort README.md](https://github.com/snort3/snort3/blob/master/README.md)
- [Understanding and Configuring Snort Rules](https://www.rapid7.com/blog/post/2016/12/09/understanding-and-configuring-snort-rules/)
- [Writing Snort Rules](http://manual-snort-org.s3-website-us-east-1.amazonaws.com/node27.html)

# Building Snort - Fedora 35
Successfully compiled using the following version of gcc:
`gcc version 11.2.1 20220127 (Red Hat 11.2.1-9) (GCC)`

Ensure the necessary applications and libraries are installed. 

**EXCEPTION**: `daq` must be built from the snort's github repository. DO NOT get `daq` from the Fedora DNF repository! It doesn't have the `daq_dlt` files that snort is looking for. 
```bash
# DO NOT INSTALL FROM DNF --> daq from https://github.com/snort3/libdaq for packet IO# 
# dnet from https://github.com/dugsong/libdnet.git for network utility functions
# flex >= 2.6.0 from https://github.com/westes/flex for JavaScript syntax parser
# g++ >= 5 or other C++14 compiler
# hwloc from https://www.open-mpi.org/projects/hwloc/ for CPU affinity management
# LuaJIT from http://luajit.org for configuration and scripting
# OpenSSL from https://www.openssl.org/source/ for SHA and MD5 file signatures, the protected_content rule option, and SSL service detection
# pcap from http://www.tcpdump.org for tcpdump style logging
# pcre from http://www.pcre.org for regular expression pattern matching
# pkgconfig from https://www.freedesktop.org/wiki/Software/pkg-config/ to locate build dependencies
# zlib from http://www.zlib.net for decompression

# Some of these may have already been installed on your system but wanted to be comprehensive.
sudo dnf install cmake libdnet flex g++ hwloc hwloc-devel libdnet-devel openssl openssl-devel libpcap-devel libpcap pcre pcre-devel luajit luajit-devel libdnet-devel libuuid-devel libuuid uuid-devel hyperscan-devel hyperscan flatbuffers flatbuffers-devel jemalloc-devel jemalloc pkgconf-pkg-config zlib zlib-devel
```

Get daq from snort3's source code repository and build it. **NOTE**: There maybe some dependencies with `daq`. On my system, this compiled on the first try.
```bash
cd /location/where/daq/source/will/be
git clone https://github.com/snort3/libdaq.git
cdi libdaq 
./bootstrap
./configure --prefix=/usr/local
make -j<NUM of THREADS>
sudo make install

# Ensure there's /usr/local/lib entry is in /etc/ld.so.conf.

# Add the library to LD_LIBRARY_PATH to your .bashrc file.
export LD_LIBRARY_PATH=/usr/lib:/usr/local/lib:${LD_LIBRARY_PATH}
source ~/.bashrc

# Run ldconfig
sudo ldconfig
```

Get the snort source and build it.
```bash
cd /location/where/snort/source/will/be
git clone https://github.com/snort3/snort3.git
cd snort3
# On my system, installation of snort3 is on /usr/local. Change --prefix accordingly.
./configure_cmake.sh --prefix=/location/of/snort3 --enable-jemalloc --enable-shell
cd build
make -j<NUM of THREADS>
sudo make install
```

Test to make sure snort is working.
```bash
cd /location/of/snort3/bin
snort -V

   ,,_     -*> Snort++ <*-
  o"  )~   Version 3.1.25.0
   ''''    By Martin Roesch & The Snort Team
           http://snort.org/contact#team
           Copyright (C) 2014-2022 Cisco and/or its affiliates. All rights reserved.
           Copyright (C) 1998-2013 Sourcefire, Inc., et al.
           Using DAQ version 3.0.6
           Using LuaJIT version 2.1.0-beta3
           Using OpenSSL 1.1.1l  FIPS 24 Aug 2021
           Using libpcap version 1.10.1 (with TPACKET_V3)
           Using PCRE version 8.45 2021-06-15
           Using ZLIB version 1.2.11
           Using FlatBuffers 2.0.0
           Using Hyperscan version 5.4.0 2021-08-11
           Using LZMA version 5.2.5
```

# Sniffer Mode
Sniff packets and send to standard output as a dump file.

|Mode |Description |
|:----|:-----------|
|-v|(verbose)|Display output on the screen|
|-e|Display link layer headers|
|-d|Display packet data payload|
|-x|Display full packet with headers in HEX format|


# Packet Logger Mode
Input and output to a log file.

|Mode |Description |
|:----|:-----------|
|-r|Use to read back the log file content using snort|
|-l (directory name)|Log to a directory as a tcpdump file format|
|-k (ASCII)|Display output as ASCII format|


# NIDS Mode
Use the specified file as config file and apply rules to process captured packets.

|Mode |Description |
|:----|:-----------|
|-c|Define configuration file path|
|-T |Use to test the configuration file including rules|


# Snort Rules Format
Rule Header + (Rule Options)
Action - Protocol - Source/Destination IP's - Source/Destination Ports - Direction of the flow

|Rule |Description |
|:----|:-----------|
|Alert Example|alert udp !10.1.1.0/24 any -> 10.2.0.0/24 any|
|Actions|alert, log, pass, activate, dynamic, drop, reject, sdrop|
|Protocols|TCP, UDP, ICMP, IP|


# Logger Mode Command Line Options

|Mode |Description |
|:----|:-----------|
|-l logdir|Log packets in tcp dump|
|-K ASCII|Log in ASCII format|

# NIDS Mode Options

|Mode |Description |
|:----|:-----------|
|Define a configuration file|-c ( Configuration file name)|
|Check the rule syntax and format for accuracy|-T -c (Configuration file name )|
|Alternate alert modes|-A (Mode : Full, Fast, None ,Console)|
|Alert to syslog|-s|
|Print alert information|-v|
|Send SMB alert to PC|-M (PC name or IP address)|
|ASCII log mode|-K|
|No logging|-N|
|Run in Background|-D|
|Listen to a specific network interface|-i|

# Snort Rule Example
`log tcp !10.1.1.0/24 any -> 10.1.1.100 (msg: "ftp access";)`

# Output Directory
Default location: `/var/snort/log`

User's output directory --> Use -l switch: `/some/other/snort/log`
