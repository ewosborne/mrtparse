mrtparse
========

mrtparse is a module to read and analyze the MRT format data.
The MRT format data can be used to export routing protocol messages, state changes, and routing information base contents, and is standardized in [RFC6396][rfc6396].
Programs like Quagga / Zebra, BIRD, OpenBGPD and PyRT can dump the MRT fotmat data.
[rfc6396]: https://tools.ietf.org/html/rfc6396

##Supported MRT types
Table_Dump(12), Table_Dump_V2(13), BGP4MP(16), BGP4MP_ET(17)

##Supported BGP capabilities
Multiprotocol Extensions for BGP-4(1), Route Refresh Capability for BGP-4(2), Outbound Route Filtering Capability(3), Graceful Restart Capability(64), Support for 4-octet AS number capability, ADD-PATH Capability(69)

##Supported BGP attributes
ORIGIN(1), AS_PATH(2), NEXT_HOP(3), MULTI_EXIT_DISC(4), LOCAL_PREF(5), ATOMIC_AGGREGATE(6), AGGREGATOR(7), COMMUNITY(8), ORIGINATOR_ID(9), CLUSTER_LIST(10), MP_REACH_NLRI(14), MP_UNREACH_NLRI(15), EXTENDED_COMMUNITIES(16), AS4_PATH(17), AS4_AGGREGATOR(18), AIGP(26), ATTR_SET(128)

##Requirements
Python2 or Python3 or PyPy or PyPy3 

##Download
###git command
    
    $ git clone https://github.com/YoshiyukiYamauchi/mrtparse.git
    
###Browser
Access [https://github.com/YoshiyukiYamauchi/mrtparse.git][mrtparse_git], and click 'Download ZIP'.
[mrtparse_git]: https://github.com/YoshiyukiYamauchi/mrtparse.git
    

##Install
    $ cd <Clone Directory>
    $ python setup.py install
    running install
    running build
    running build_py
    running install_lib
    copying build/lib/mrtparse.py -> /Library/Python/2.7/site-packages
    byte-compiling /Library/Python/2.7/site-packages/mrtparse.py to mrtparse.pyc
    running install_egg_info
    Writing /Library/Python/2.7/site-packages/mrtparse-0.8-py2.7.egg-info


##Usage
    
    from mrtparse import *
    
or
    
    import mrtparse
    
##Programming
First, import the module.
    
    from mrtparse import *
    
And pass a MRT format data as a filepath string or file object to a class Reader().   
It is also supported gzip and bzip2 format.  
You can retrieve each entry from the returned object using a loop and then process it.  

    
    d = Reader(f)
    for m in d:
        <statements>
    

##Example
These scripts are included in 'examples' directory.
###print_all.py
####Description
This script displays the contents of a MRT format file.
####Usage
    print_all.py <path to the file>
####Result
    ---------------------------------------------------------------
    MRT Header
        Timestamp: 1392828028(2014-02-20 01:40:28)
        Type: 16(BGP4MP)
        Subtype: 5(BGP4MP_STATE_CHANGE_AS4)
        Length: 24
    BGP4MP_STATE_CHANGE_AS4
        Peer AS Number: 100
        Local AS Number: 64512
        Interface Index: 0
        Address Family: 1(AFI_IPv4)
        Peer IP Address: 192.168.1.21
        Local IP Address: 192.168.1.100
        Old State: 5(OpenConfirm)
        New State: 6(Established)
    ---------------------------------------------------------------
    MRT Header
        Timestamp: 1392828028(2014-02-20 01:40:28)
        Type: 16(BGP4MP)
        Subtype: 4(BGP4MP_MESSAGE_AS4)
        ...

If error occurred, it displays data in byte as below.

    ---------------------------------------------------------------
    MRT Header
        Timestamp: 1442843462(2015-09-21 22:51:02)
        Type: 16(BGP4MP)
        Subtype: 5(BGP4MP_STATE_CHANGE_AS4)
        Length: 12
    MRT Data Error: Unknown AFI 8
        00 00 00 00 00 00 00 00  00 01 00 08

    ---------------------------------------------------------------
    MRT Header
        Timestamp: 1443251615(2015-09-26 16:13:35)
        Type: 16(BGP4MP)
        Subtype: 2(BGP4MP_ENTRY)
        Length: 80
    MRT Data Error: Unsupported BGP4MP Subtype 2(BGP4MP_ENTRY)
        fd e8 fd e8 00 00 00 01  c0 a8 01 66 c0 a8 01 0a 
        00 00 00 01 56 06 43 5f  00 01 01 04 c0 a8 00 0b 
        10 c0 a8 00 2b 40 01 01  00 40 02 04 02 01 fd f3 
        40 05 04 00 00 00 64 c0  07 08 00 00 fd e8 c0 a8 
        00 0b 80 09 04 c0 a8 00  0b 80 0a 04 c0 a8 00 0a

        
###mrt2exabgp.py (formerly exabgp_conf.py)
####Description
This script converts MRT format to [ExaBGP][exabgp_git] config/API format and displays it.
If you want to know how to use ExaBGP API, please read [the wiki][wiki]. 
[exabgp_git]: https://github.com/Exa-Networks/exabgp
[wiki]: https://github.com/YoshiyukiYamauchi/mrtparse/wiki
####Usage
    usage: mrt2exabgp.py [-h] [-r ROUTER_ID] [-l LOCAL_AS] [-p PEER_AS]
                         [-L LOCAL_ADDR] [-n NEIGHBOR] [-4 [NEXT_HOP]]
                         [-6 [NEXT_HOP]] [-a] [-A] [-G [NUM]]
                         path_to_file
    
    This script converts to ExaBGP format config.
    
    positional arguments:
      path_to_file   specify path to MRT format file
    
    optional arguments:
      -h, --help     show this help message and exit
      -r ROUTER_ID   specify router-id (default: 192.168.0.1)
      -l LOCAL_AS    specify local AS number (default: 64512)
      -p PEER_AS     specify peer AS number (default: 65000)
      -L LOCAL_ADDR  specify local address (default: 192.168.1.1)
      -n NEIGHBOR    specify neighbor address (default: 192.168.1.100)
      -4 [NEXT_HOP]  convert IPv4 entries and use IPv4 next-hop if specified
      -6 [NEXT_HOP]  convert IPv6 entries and use IPv6 next-hop if specified
      -a             convert all entries (default: convert only first entry per
                     one prefix)
      -A             convert to ExaBGP API format
      -G [NUM]       convert to ExaBGP API format and group updates with the same
                     attributes for each spceified the number of prefixes
                     (default: 1000000)

####Result (Config format)
    neighbor 192.168.1.1 {
        router-id 192.168.0.2;
        local-address 192.168.1.2;
        local-as 64512;
        peer-as 65000;
        graceful-restart;
        aigp enable;
    
        static {
            route 1.0.0.0/24 origin IGP as-path [57821 12586 13101 15169 ] community [12586:147 12586:13000 64587:13101] next-hop 192.168.1.254;
            route 1.0.4.0/24 origin IGP as-path [57821 6939 4826 56203 ] next-hop 192.168.1.254;
            route 1.0.5.0/24 origin IGP as-path [57821 6939 4826 56203 ] next-hop 192.168.1.254;
            route 1.0.6.0/24 origin IGP as-path [57821 6939 4826 56203 ] next-hop 192.168.1.254;
            route 1.0.7.0/24 origin IGP as-path [57821 6939 4826 56203 56203 56203 ] next-hop 192.168.1.254;
            route 1.0.64.0/18 origin IGP as-path [57821 6939 4725 4725 7670 7670 7670 18144 ] atomic-aggregate aggregator (18144:219.118.225.189) next-hop 192.168.1.254;
            route 1.0.128.0/17 origin IGP as-path [57821 12586 3257 38040 9737 ] atomic-aggregate aggregator (9737:203.113.12.254) community [12586:145 12586:12000 64587:3257] next-hop 192.168.1.254;
            route 1.0.128.0/18 origin IGP as-path [57821 12586 3257 38040 9737 ] atomic-aggregate aggregator (9737:203.113.12.254) community [12586:145 12586:12000 64587:3257] next-hop 192.168.1.254;
            ...
        }
    }

####Result (API format)
    announce route 1.0.0.0/24 origin IGP as-path [57821 12586 13101 15169 ] community [12586:147 12586:13000 64587:13101] next-hop 192.168.1.254
    announce route 1.0.4.0/24 origin IGP as-path [57821 6939 4826 56203 ] next-hop 192.168.1.254
    announce route 1.0.5.0/24 origin IGP as-path [57821 6939 4826 56203 ] next-hop 192.168.1.254
    announce route 1.0.6.0/24 origin IGP as-path [57821 6939 4826 56203 ] next-hop 192.168.1.254
    announce route 1.0.7.0/24 origin IGP as-path [57821 6939 4826 56203 56203 56203 ] next-hop 192.168.1.254
    announce route 1.0.64.0/18 origin IGP as-path [57821 6939 4725 4725 7670 7670 7670 18144 ] atomic-aggregate aggregator (18144:219.118.225.189) next-hop 192.168.1.254
    announce route 1.0.128.0/17 origin IGP as-path [57821 12586 3257 38040 9737 ] atomic-aggregate aggregator (9737:203.113.12.254) community [12586:145 12586:12000 64587:3257] next-hop 192.168.1.254
    announce route 1.0.128.0/18 origin IGP as-path [57821 12586 3257 38040 9737 ] atomic-aggregate aggregator (9737:203.113.12.254) community [12586:145 12586:12000 64587:3257] next-hop 192.168.1.254
    ...

####Result (API grouping format)
    announce attribute origin IGP as-path [57821 6939 4826 56203 ] next-hop 192.168.1.254 nlri 1.0.4.0/24 1.0.5.0/24 1.0.6.0/24 103.2.176.0/24 103.2.177.0/24 103.2.178.0/24 103.2.179.0/24
    announce attribute origin IGP as-path [57821 6939 4826 56203 56203 56203 ] next-hop 192.168.1.254 nlri 1.0.7.0/24
    announce attribute origin IGP as-path [57821 6939 4725 4725 7670 7670 7670 18144 ] atomic-aggregate aggregator (18144:219.118.225.189) next-hop 192.168.1.254 nlri 1.0.64.0/18 58.183.0.0/16 222.231.64.0/18
    announce attribute origin IGP as-path [57821 12586 3257 38040 9737 ] atomic-aggregate aggregator (9737:203.113.12.254) community [12586:145 12586:12000 64587:3257] next-hop 192.168.1.254 nlri 1.0.128.0/17 1.0.128.0/18 1.0.192.0/18 1.2.128.0/17 1.4.128.0/17 1.4.128.0/18 1.179.128.0/17 101.51.0.0/16 101.51.64.0/18 113.53.0.0/16 113.53.0.0/18 118.172.0.0/16 118.173.0.0/16 118.173.192.0/18 118.174.0.0/16 118.175.0.0/16 118.175.0.0/18 125.25.0.0/16 125.25.128.0/18 180.180.0.0/16 182.52.0.0/16 182.52.0.0/17 182.52.128.0/18 182.53.0.0/16 182.53.0.0/18 182.53.192.0/18
    announce attribute origin IGP as-path [4608 1221 4637 4651 9737 23969 ] next-hop 192.168.1.254 nlri 1.0.128.0/24
    announce attribute origin IGP as-path [57821 12586 3257 1299 38040 9737 ] atomic-aggregate aggregator (9737:203.113.12.254) community [12586:145 12586:12000 64587:3257] next-hop 192.168.1.254 nlri 1.0.160.0/19 1.0.224.0/19 118.173.64.0/19 118.173.192.0/19 118.174.128.0/19 118.174.192.0/19 118.175.160.0/19 125.25.0.0/19 125.25.128.0/19 182.53.0.0/19 203.113.0.0/19 203.113.96.0/19
    announce attribute origin IGP as-path [57821 12586 3257 4134 ] community [12586:145 12586:12000 64587:3257] next-hop 192.168.1.254 nlri 1.1.8.0/24 36.106.0.0/16 36.108.0.0/16 36.109.0.0/16 101.248.0.0/16 106.0.4.0/22 106.7.0.0/16 118.85.204.0/24 118.85.215.0/24 120.88.8.0/21 122.198.64.0/18 171.44.0.0/16 183.91.56.0/24 183.91.57.0/24 202.80.192.0/22 221.231.151.0/24
    announce attribute origin IGP as-path [57821 12586 13101 15412 17408 58730 ] community [12586:147 12586:13000 64587:13101] next-hop 192.168.1.254 nlri 1.1.32.0/24 1.2.1.0/24 1.10.8.0/24 14.0.7.0/24 27.34.239.0/24 27.109.63.0/24 36.37.0.0/24 42.0.8.0/24 49.128.2.0/24 49.246.249.0/24 101.102.104.0/24 106.3.174.0/24 118.91.255.0/24 123.108.143.0/24 180.200.252.0/24 183.182.9.0/24 202.6.6.0/24 202.12.98.0/24 202.85.202.0/24 202.131.63.0/24 211.155.79.0/24 211.156.109.0/24 218.98.224.0/24 218.246.137.0/24
    ...


###slice.py
####Description
This script outputs the following data of a MRT format file.  
1. The data for the interval of the specified seconds from the specified start time to the specified end time.  
2. The data from the specified start time to the specified end time.  
3. The data for the interval of the specified seconds.  
####Usage
    usage: slice.py [-h] [-s START_TIME] [-e END_TIME] [-i INTERVAL] [-c {gz,bz2}]
                    path_to_file
    
    This script slices MRT format data.
    
    positional arguments:
      path_to_file    specify path to MRT format file
    
    optional arguments:
      -h, --help     show this help message and exit
      -s START_TIME  specify start time in format YYYY-MM-DD HH:MM:SS
      -e END_TIME    specify end time in format YYYY-MM-DD HH:MM:SS
      -i INTERVAL    specify interval in seconds
      -c {gz,bz2}    specify compress type (gz, bz2)

####Result
    # slice.py -s '2015-04-26 03:26:00' -e '2014-04-26 03:27:00' -i 10 -c bz2 -f latest-update.gz
    # ls
    latest-update-20150426-032600.bz2
    latest-update-20150426-032610.bz2
    latest-update-20150426-032620.bz2
    latest-update-20150426-032630.bz2
    latest-update-20150426-032640.bz2
    latest-update-20150426-032650.bz2


###summary.py
####Description
This script displays the summary of a MRT format file.
####Usage
    summary.py <path to the file>
####Result
    [2014-08-11 03:45:00 - 2014-08-11 03:49:59]
    BGP4MP:                             5973
        BGP4MP_MESSAGE:                   34
            UPDATE:                       24
            KEEPALIVE:                    10
        BGP4MP_MESSAGE_AS4:             5896
            UPDATE:                     5825
            KEEPALIVE:                    71
        BGP4MP_STATE_CHANGE_AS4:          43
            Idle:                          1
            Connect:                      20
            Active:                       18
            OpenSent:                      4


###mrt2bgpdump.py
####Description
This script converts to [bgpdump][bgpdump] format.
[bgpdump]: https://bitbucket.org/ripencc/bgpdump/wiki/Home
####Usage
    usage: mrt2bgpdump.py [-h] [-m] [-M] [-O [file]] [-s] [-v] [-t {dump,change}]
                          [-p]
                          path_to_file
    
    This script converts to bgpdump format.
    
    positional arguments:
      path_to_file      specify path to MRT format file
    
    optional arguments:
      -h, --help        show this help message and exit
      -m                one-line per entry with unix timestamps
      -M                one-line per entry with human readable timestamps(default
                        format)
      -O [file]         output to a specified file
      -s                output to STDOUT(default output)
      -v                output to STDERR
      -t {dump,change}  timestamps for RIB dumps reflect the time of the dump or
                        the last route modification(default: dump)
      -p                show packet index at second position
####Result
    BGP4MP|0|1438386900|A|193.0.0.56|3333|204.80.242.0/24|3333 1273 7922 33667 54169 54169 54169 54169 54169 54169 54169 54169|IGP|193.0.0.56|0|0|1273:21000|NAG||
    BGP4MP|1|1438386900|A|2405:fc00::6|37989|2001:4c0:2001::/48|37989 4844 2914 174 855|IGP|2405:fc00::6|0|0||NAG||
    BGP4MP|1|1438386900|A|2405:fc00::6|37989|2001:4c0:6002::/48|37989 4844 2914 174 855|IGP|2405:fc00::6|0|0||NAG||
    BGP4MP|2|1438386900|A|146.228.1.3|1836|189.127.0.0/21|1836 174 12956 262589 27693|IGP|146.228.1.3|0|0|1836:110 1836:6000 1836:6031|NAG|27693 189.127.15.253|
    BGP4MP|4|1438386900|A|2405:fc00::6|37989|2406:e400:1a::/48|37989 4844 7642|INCOMPLETE|2405:fc00::6|0|0||NAG||
    BGP4MP|5|1438386900|A|2001:8e0:0:ffff::9|8758|2c0f:fe90::/32|8758 174 2914 30844 37105 37105 37105 36943|IGP|2001:8e0:0:ffff::9|0|0|174:21100 174:22005 8758:110 8758:301|NAG||
    BGP4MP|6|1438386900|A|213.200.87.254|3257|187.110.144.0/20|3257 174 16735 27693 53117|IGP|213.200.87.254|0|10|3257:8093 3257:30235 3257:50002 3257:51100 3257:51102|NAG||
    BGP4MP|7|1438386900|A|213.200.87.254|3257|187.95.16.0/20|3257 174 16735 27693 53081|IGP|213.200.87.254|0|10|3257:8063 3257:30252 3257:50002 3257:51300 3257:51302|NAG||
    BGP4MP|8|1438386900|A|213.200.87.254|3257|189.127.208.0/21|3257 174 16735 27693 28235|IGP|213.200.87.254|0|10|3257:8093 3257:30235 3257:50002 3257:51100 3257:51102|NAG||
    BGP4MP|8|1438386900|A|213.200.87.254|3257|189.127.216.0/21|3257 174 16735 27693 28235|IGP|213.200.87.254|0|10|3257:8093 3257:30235 3257:50002 3257:51100 3257:51102|NAG||
    ...


##Authors
Tetsumune KISO <t2mune@gmail.com>  
Yoshiyuki YAMAUCHI <info@greenhippo.co.jp>  
Nobuhiro ITOU <js333123@gmail.com>  

License
----------
Licensed under the [Apache License, Version 2.0][Apache]  
Copyright &copy; 2015 [greenHippo, LLC.][greenHippo]  
[Apache]: http://www.apache.org/licenses/LICENSE-2.0
[greenHippo]: http://greenhippo.co.jp
