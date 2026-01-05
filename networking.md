# Linux Network Administration


## Understanding Linux Network Interfaces
- Different versions of Linux may name network interfaces differently, In general, just about all Linux operating systems will have at least two network interfaces. They are:
- `Loopback`: THe *loopback (lo)* interface will have an IP address of **127.0.0.1**, which represents the host itself. Suppose you want to open a web page running on the same Linux server you are on. You could open *http://127.0.0.1* in your web browser. That IP address won't accessible over the network.
- `Ethernet`: The *ethernet 0 (eth0)* interface is typically the connection to the local network.
  - Even you are running Linux in a virtual machine, you'll still have an eth0 interface that connects to the physical network of the host.
  - Most commonly, you should ensure that eth0 is in a **UP** state and has an IP address so that you can communicate with the local network and likely over the internet.

- The linux command to configure network interfaces/devices/links is `ip link`. In the following example, you can see how `ip link` shows teo different interfaces, their status and their MAC addresses associated with each one.

    ```
    vagrant@ubuntulvmlab0101:~$ ip link
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
        link/ether 08:00:27:d9:fa:93 brd ff:ff:ff:ff:ff:ff
        altname enp0s8
    ```
- The `ip link` command is also used to configure network interfaces. For example, you can change the status of interfaces with `ip link set [dev] {up | down }`.

### Example using `ip link`
- Bring an interface up 
    ```
    vagrant@ubuntulvmlab0101:~$ sudo ip link set eth0 up
    vagrant@ubuntulvmlab0101:~$ 
    vagrant@ubuntulvmlab0101:~$ ip link show eth0
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
        link/ether 08:00:27:d9:fa:93 brd ff:ff:ff:ff:ff:ff
        altname enp0s8
    ```
- Bring an interface down

    ```
    vagrant@ubuntulvmlab0101:~$ sudo ip link set lo down
    vagrant@ubuntulvmlab0101:~$ ip link show lo
    1: lo: <LOOPBACK> mtu 65536 qdisc noqueue state DOWN mode DEFAULT group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    ```
- Changing the MTU (maximum transmission unit)
    ```
    vagrant@ubuntulvmlab0101:~$ sudo ip link set eth0 mtu 1400
    vagrant@ubuntulvmlab0101:~$ ip link show eth0
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
        link/ether 08:00:27:d9:fa:93 brd ff:ff:ff:ff:ff:ff
        altname enp0s8
    ```
- Renaming a network interface
    ```
    vagrant@ubuntulvmlab0101:~$ sudo ip link set lo down
    vagrant@ubuntulvmlab0101:~$ sudo ip link set lo name loopback
    vagrant@ubuntulvmlab0101:~$ sudo ip link set  loopback up
    vagrant@ubuntulvmlab0101:~$ 
    vagrant@ubuntulvmlab0101:~$ ip link show loopback
    1: loopback: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    ```

- Create a dummy interface
    
    ```
    vagrant@ubuntulvmlab0101:~$ sudo ip link add eth-dummy type dummy
    vagrant@ubuntulvmlab0101:~$ ip link show eth-dummy
    3: eth-dummy: <BROADCAST,NOARP> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
        link/ether 32:cb:55:dd:ef:71 brd ff:ff:ff:ff:ff:ff
    ```
- Delete a network interface (only works for virtual not physical NICs)  

    ```
    vagrant@ubuntulvmlab0101:~$ sudo ip link delete eth-dummy
    vagrant@ubuntulvmlab0101:~$ 
    vagrant@ubuntulvmlab0101:~$ ip link | grep eth-dummy
    ```
- Create a VLAN interface

    ```
    vagrant@ubuntulvmlab0101:~$ sudo ip link add link eth0 name eth0.100 type vlan id 100
    vagrant@ubuntulvmlab0101:~$ sudo ip link set eth0.100  up
    vagrant@ubuntulvmlab0101:~$ ip link show eth0.100
    4: eth0.100@eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc noqueue state UP mode DEFAULT group default qlen 1000
        link/ether 08:00:27:d9:fa:93 brd ff:ff:ff:ff:ff:ff
    ```

- Create a bridge interface
    ```
    vagrant@ubuntulvmlab0101:~$ sudo ip link add br0 type bridge
    vagrant@ubuntulvmlab0101:~$ sudo ip link set  br0 up
    vagrant@ubuntulvmlab0101:~$ 
    vagrant@ubuntulvmlab0101:~$ ip link show br0
    5: br0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default qlen 1000
        link/ether 26:3d:b0:c1:7d:77 brd ff:ff:ff:ff:ff:ff
    ```
- NOTE: The `ip link` changes are NOT persistent, 
  - After reboot: interfaces revert, MTU resets and Dummy/veth/VLAN interfaces disappear
  - For persistence use: `/etc/netplan` (ubuntu), `/etc/sysconfig/network-scripts` (RHEL), NetworkManager, systemd-networkd

### Predictable network interface naming convention
- If you've been in IT for any length of time, you know that the only constant is change, and that applies to network interface naming conventions as well.
- Up until very recently, the only naming convention you had to worry about was the one that was just presented.
- However, with systems featuring systemd v197 or later, common network names such as "eth0" will be assigned more predictable names based on what is the known as the *predictable network interface* naming convention.
- So, rather than an interface named eth0, you may have one named ens3 or ens0f1.
- Ironically under older versions of systemd, there wa predictability about the existence of eth0. Under the Predictable Network Interface naming guidelines, however, there isn't guaranteed standard network interface names across systems.
- The new scheme does have a number of benefits, the most important of which is the physical interface to interface name association will survive reboots, even if you add hardware.
- Under the old system, if you added a network adapter, you ran the risk of automatically changing interface names, which had serious consequences for configuration.

---

## MAC Addresses
- A media access control (MAC) address is the unique identifier assigned to a network interface at layer-2 -- the Data layer of the OSI model.
- A network interface always has a MAC address -- often referred to as the *hardware* address -- even if it does not have an IP address.
- MAC addresses are assigned at the time that a network adapter is manufactured or, if it;s a virtualized network adapter, the time that the adapter is created and appears as six groups of two hexadecimal digits each.

    ```
    vagrant@ubuntulvmlab0101:~$ ip link show eth0
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
        link/ether 08:00:27:d9:fa:93 brd ff:ff:ff:ff:ff:ff
        altname enp0s8
    ```
- On the Ethernet interface, eth0, shown above, the MAC address is also called *link* or *ether address*. In the abobe example, the MAC address is *08:00:27:d9:fa:93*

---

## IP Addressing
- They are unique on the same network, every device has at least one, and addresses typically fall somewhere between `1.1.1.1` and `255.255.255.255`.
- To view IP addresses, use the `ip address` command or just `ip addr` or `ip a`

    ```
    vagrant@ubuntulvmlab0101:~$ ip addr
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host 
           valid_lft forever preferred_lft forever
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UP group default qlen 1000
        link/ether 08:00:27:d9:fa:93 brd ff:ff:ff:ff:ff:ff
        altname enp0s8
        inet 10.0.2.15/24 metric 100 brd 10.0.2.255 scope global dynamic eth0
           valid_lft 83519sec preferred_lft 83519sec
        inet6 fd17:625c:f037:2:a00:27ff:fed9:fa93/64 scope global dynamic mngtmpaddr noprefixroute 
           valid_lft 86086sec preferred_lft 14086sec
        inet6 fe80::a00:27ff:fed9:fa93/64 scope link 
           valid_lft forever preferred_lft forever
    ```

- As you can see the IP address of the loopback interface is *127.0.0.1*. What is the IP address of eth0 interface in this case? The eth0 "inet" address (IPV4 address) is *10.0.2.15*, and it's a dynamic address, received via DHCP.
- When it comes to linux networking tools, there is one that just about every one heard of, and that is **ping**.
- It sends out an *Internet Control Message Protocol* (ICMP) packet across the network and notifies you whether there is a response.
- If a host is up and able to communicate on the network, an ICMP response will be returned. If, however a host is not rechable, you will get a notice that the host was unreachable ot timed out.

    ```
    #### success case
    
    vagrant@ubuntulvmlab0101:~$ ping -c 5 google.com
    PING google.com (142.250.205.78) 56(84) bytes of data.
    64 bytes from pnmaaa-ar-in-f14.1e100.net (142.250.205.78): icmp_seq=1 ttl=255 time=36.5 ms
    64 bytes from pnmaaa-ar-in-f14.1e100.net (142.250.205.78): icmp_seq=2 ttl=255 time=15.8 ms
    64 bytes from pnmaaa-ar-in-f14.1e100.net (142.250.205.78): icmp_seq=3 ttl=255 time=26.0 ms
    64 bytes from pnmaaa-ar-in-f14.1e100.net (142.250.205.78): icmp_seq=4 ttl=255 time=13.1 ms
    64 bytes from pnmaaa-ar-in-f14.1e100.net (142.250.205.78): icmp_seq=5 ttl=255 time=20.6 ms
    
    --- google.com ping statistics ---
    5 packets transmitted, 5 received, 0% packet loss, time 4012ms
    rtt min/avg/max/mdev = 13.128/22.394/36.528/8.313 ms
    
    
    #### failure case
    
    vagrant@ubuntulvmlab0101:~$ ping -c 5 192.168.10.11
    PING 192.168.10.11 (192.168.10.11) 56(84) bytes of data.
    
    --- 192.168.10.11 ping statistics ---
    5 packets transmitted, 0 received, 100% packet loss, time 4095ms
    
    ```
- Another common linux network troubleshooting tool is `traceroute`. Traceroute probes the network between the local system and a destination, gathering information about each IP router in the path.
- The `traceroute` command is useful when you think there may be a network issue, such as host down along a path or a slow response from one of the intermediate nodes, and you want to find out which node is creating the problem.

    ```
    vagrant@ubuntulvmlab0101:~$ traceroute www.google.com
    traceroute to www.google.com (142.251.223.164), 30 hops max, 60 byte packets
     1  _gateway (10.0.2.2)  0.183 ms  0.063 ms  0.110 ms
     2  * * *
     3  * * *
     4  * * *
     5  * * *
     6  * * *
     7  * * *
     8  * * *
     9  * * *
    10  * * *
    11  * * *
    12  * * *
    13  * * *
    14  * * *
    15  * * *
    16  * * *
    17  * * *
    18  * * *
    19  * * *
    20  * * *
    21  * * *
    22  * * *
    23  * * *
    24  * * *
    25  * * *
    26  * * *
    27  * * *
    28  * * *
    29  * * *
    30  * * *
    ```
---

## DHCP
- What if you have dozens, hundreds, or thousands of computers on your network? It would be incredibly time-consuming to manually assign IP address and to actually track which machines have which IP address. That's where *dynamic host configuration protocol* comes in. 
- DHCP is used to obtain an IP address when a host or device first comes on the network.
- DHCP is commonly used for client systems or devices that don't experience any side effects from a periodically changing IP address.
- On server systems, administrators either manually configure static IP addresses, or they create what are knows as static DHCP reservations that are tied to the MAC addresses of the network adapter.
- These static reservations ensure that the network adapter will get the same IP addresses every time it restarts.
- Here is how the typical DHCP process works:
  - When a computer starts up, it sends a DHCP request out on the network
  - Assuming a DHCP server is present, a DHCP server responds with the *IP address configuration* for that device.
  - That IP address is marked as reserved so that it's not accidentally assigned to some other device.
- The DHCP server responds with *IP address configuration*, which means the configuration returned to a requesting client contains, at minimum, the IP address, the IP subnet mask, the IP default gateway, and DNS server details. Most end user devices are configured to use DHCP.
- The local configuration file for the DHCP client (called *dhclient*) is at `/etc/dhcp/dhclient.conf`. This is a configuration file that dictates to Linux how it will receive IP configuration information from a DHCP server.
- To check the status on the DHCP client, you can cat the syslog and grep foir dhcp like below

    ```
    vagrant@ubuntulvmlab0101:~$ grep -i dhcp /var/log/syslog 
    Jan  1 08:57:30 vagrant systemd-networkd[648]: eth0: DHCPv4 address 10.0.2.15/24 via 10.0.2.2
    Jan  1 08:57:30 vagrant kernel: [    6.239734] audit: type=1400 audit(1767257845.624:5): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/lib/NetworkManager/nm-dhcp-client.action" pid=621 comm="apparmor_parser"
    Jan  1 08:57:30 vagrant kernel: [    6.240777] audit: type=1400 audit(1767257845.624:6): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/lib/NetworkManager/nm-dhcp-helper" pid=621 comm="apparmor_parser"
    Jan  1 08:57:38 vagrant systemd-networkd[648]: eth0: DHCPv6 lease lost
    Jan  1 08:57:39 vagrant systemd-networkd[1474]: eth0: DHCPv4 address 10.0.2.15/24 via 10.0.2.2
    Jan  1 09:02:35 vagrant systemd-networkd[1474]: eth0: DHCP lease lost
    Jan  1 09:02:35 vagrant systemd-networkd[1474]: eth0: DHCPv6 lease lost
    Jan  1 09:02:37 vagrant systemd-networkd[1474]: eth0: DHCPv4 address 10.0.2.15/24 via 10.0.2.2
    ```

---

## DNS
- Computers that connect to each other using TCP/IP, talk with other using IP addresses; however, it would be really painful to have to remember the IP address of everything you want to connect to.
- Imagine having to recall the IP address of Google each time you wanted to search the web.
- *Domain name system (DNS)* is used to map IP addresses to names. Everyone is familiar with using their web browser, entering a friendly name like google.com or apple.com, and being taken to the company's website without ever having to type an IP address.
- It is DNS behind the scenes that is mapping the friendly name to an IP addresses by doing a DNS lookup. 
- To find out if yur linux host is using DNS, we will be running through some troubleshooting commands, such as `dig` and `nslookup`
- The basics of DNS in Linux are this:
  - A local file called /etc/hosts is used for the first point of lookup for any host name prior to going out to a DNS server on the network. If the same is found there, no further searches are performed, As the superuser, you have the option to edit the hosts file and configure a static name to IP addresses mapping.
  - The /etc/resolv.conf file shows the local domains to be searched and what server names to use for DNS resolution.
- A sample resolve.conf file

    ```
    vagrant@ubuntulvmlab0101:~$ cat /etc/resolv.conf 
    # This is /run/systemd/resolve/stub-resolv.conf managed by man:systemd-resolved(8).
    # Do not edit.
    #
    # This file might be symlinked as /etc/resolv.conf. If you're looking at
    # /etc/resolv.conf and seeing this text, you have followed the symlink.
    #
    # This is a dynamic resolv.conf file for connecting local clients to the
    # internal DNS stub resolver of systemd-resolved. This file lists all
    # configured search domains.
    #
    # Run "resolvectl status" to see details about the uplink DNS servers
    # currently in use.
    #
    # Third party programs should typically not access this file directly, but only
    # through the symlink at /etc/resolv.conf. To manage man:resolv.conf(5) in a
    # different way, replace this symlink by a static file or a different symlink.
    #
    # See man:systemd-resolved.service(8) for details about the supported modes of
    # operation for /etc/resolv.conf.
    
    nameserver 127.0.0.53
    options edns0 trust-ad
    search .
    ```
- When it comes to troubleshooting DNS, you should be aware of the following important tools:
  - `dig`: The *domain internet roper*, or dig, performs verbose DNS lookups and is great for troubleshooting DNS issues.
  - `getent ahosts`: The getent tool with the ahosts option enumerates name service switch files, specifically for host entries.
  - `nslookup`: The name server lookup, performs a variety of different DNS server lookups: mail server lookups, reverse lookups, and more. It's commonly used to look up the IP address of a host.

---

## Network Statistics and Counters
- When performing network troubleshooting, it's always good to gather some statistics to answer questions like:
  - Is the network interface even transmitting any data? 
  - Is the interface taking errors? 
  - What process is sending all that traffic?
- Here is an example of the `ip -s link` command showing us the statistics for our network links:

    ```
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        RX:  bytes packets errors dropped  missed   mcast           
             23414     214      0       0       0       0 
        TX:  bytes packets errors dropped carrier collsns           
             23414     214      0       0       0       0 
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
        link/ether 08:00:27:d9:fa:93 brd ff:ff:ff:ff:ff:ff
        RX:  bytes packets errors dropped  missed   mcast           
         158007391  111570      0       0       0     591 
        TX:  bytes packets errors dropped carrier collsns           
           1448974   19083      0       0       0       0 
        altname enp0s8
    ```
- Here an example of the `netstat` command showing us what our active process are that have the network interface open:

    ```
    vagrant@ubuntulvmlab0101:~$ netstat
    Active Internet connections (w/o servers)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State      
    tcp        0      0 ubuntulvmlab0101:ssh    _gateway:58651          ESTABLISHED
    tcp        0      0 ubuntulvmlab0101:38848  ubuntu-ports-mirro:http TIME_WAIT  
    Active UNIX domain sockets (w/o servers)
    Proto RefCnt Flags       Type       State         I-Node   Path
    unix  2      [ ]         DGRAM                    23965    /run/user/1000/systemd/notify
    unix  3      [ ]         DGRAM      CONNECTED     18073    /run/systemd/notify
    unix  2      [ ]         DGRAM                    18091    /run/systemd/journal/syslog
    unix  9      [ ]         DGRAM      CONNECTED     18100    /run/systemd/journal/dev-log
    unix  8      [ ]         DGRAM      CONNECTED     18102    /run/systemd/journal/socket
    unix  3      [ ]         STREAM     CONNECTED     19857    
    unix  3      [ ]         STREAM     CONNECTED     24885    /run/systemd/journal/stdout
    .....
    ```

- Here is an example of the `netstat -l` command that shows us the active listening services on this host:

    ```
    vagrant@ubuntulvmlab0101:~$ netstat -l
    Active Internet connections (only servers)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State      
    tcp        0      0 localhost:domain        0.0.0.0:*               LISTEN     
    tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN     
    tcp6       0      0 [::]:ssh                [::]:*                  LISTEN     
    udp        0      0 localhost:domain        0.0.0.0:*                          
    udp        0      0 ubuntulvmlab0101:bootpc 0.0.0.0:*                          
    raw6       0      0 [::]:ipv6-icmp          [::]:*                  7          
    Active UNIX domain sockets (only servers)
    Proto RefCnt Flags       Type       State         I-Node   Path
    unix  2      [ ACC ]     STREAM     LISTENING     26182    /var/snap/lxd/common/lxd/unix.socket
    unix  2      [ ACC ]     STREAM     LISTENING     23968    /run/user/1000/systemd/private
    unix  2      [ ACC ]     STREAM     LISTENING     18090    @/org/kernel/linux/storage/multipathd
    unix  2      [ ACC ]     STREAM     LISTENING     23975    /run/user/1000/bus
    unix  2      [ ACC ]     STREAM     LISTENING     23977    /run/user/1000/gnupg/S.dirmngr
    unix  2      [ ACC ]     STREAM     LISTENING     23979    /run/user/1000/gnupg/S.gpg-agent.browser
    unix  2      [ ACC ]     STREAM     LISTENING     23981    /run/user/1000/gnupg/S.gpg-agent.extra
    unix  2      [ ACC ]     STREAM     LISTENING     23983    /run/user/1000/gnupg/S.gpg-agent.ssh
    unix  2      [ ACC ]     STREAM     LISTENING     23985    /run/user/1000/gnupg/S.gpg-agent
    unix  2      [ ACC ]     STREAM     LISTENING     23987    /run/user/1000/pk-debconf-socket
    unix  2      [ ACC ]     STREAM     LISTENING     23989    /run/user/1000/snapd-session-agent.socket
    unix  2      [ ACC ]     STREAM     LISTENING     18076    /run/systemd/private
    unix  2      [ ACC ]     STREAM     LISTENING     18078    /run/systemd/userdb/io.systemd.DynamicUser
    unix  2      [ ACC ]     STREAM     LISTENING     18079    /run/systemd/io.system.ManagedOOM
    unix  2      [ ACC ]     STREAM     LISTENING     18088    /run/lvm/lvmpolld.socket
    unix  2      [ ACC ]     STREAM     LISTENING     18093    /run/systemd/fsck.progress
    unix  2      [ ACC ]     STREAM     LISTENING     18104    /run/systemd/journal/stdout
    unix  2      [ ACC ]     SEQPACKET  LISTENING     18107    /run/udev/control
    unix  2      [ ACC ]     STREAM     LISTENING     18368    /run/systemd/journal/io.systemd.journal
    unix  2      [ ACC ]     STREAM     LISTENING     26173    /var/snap/lxd/common/lxd-user/unix.socket
    unix  2      [ ACC ]     STREAM     LISTENING     19912    /run/systemd/resolve/io.systemd.Resolve
    unix  2      [ ACC ]     STREAM     LISTENING     20254    @ISCSIADM_ABSTRACT_NAMESPACE
    unix  2      [ ACC ]     STREAM     LISTENING     20252    /run/dbus/system_bus_socket
    unix  2      [ ACC ]     STREAM     LISTENING     20261    /run/snapd.socket
    unix  2      [ ACC ]     STREAM     LISTENING     20263    /run/snapd-snap.socket
    unix  2      [ ACC ]     STREAM     LISTENING     20265    /run/uuidd/request
    ```
  
---

## How to configure Network Interfaces
- When it comes to making any changes in Linux, keep in mind that you can make two types of changes:
  - Changes that are immediately effective but are *non-persistent* (they won't survive restart)
  - Changes that are effective after the next restart of the OS, known as *persistent changes*
- To make an immediately effective change in your IP configuration, you use the `ip` command with its variety of command options such as `link`, `route`, and `address`.
- To use the ip command set, you will have to have the iproute2 package installed, usually installed by default.
- Here we can use the `ip address` command like this:
  - `ip addr add 10.10.10.10/8 dev eth0`
- However, once the Linux machine is restarted, the default IP address will be back on interface eth0.
- To make this IP address change persistent on Debian or Ubuntu system, you need to edit the file `/etc/network/interfaces` and add the configuration for eth0.
- If you are using CentOS or RHEL, the same configuration is found in the `/etc/sysconfig/network-scripts` directory.
- To make the IP address change take effect, you can either reboot the host or use the `ifdown/ifup` commands.

### ifdown and ifup commands
- The `ifdown` and `ifup` commands are used to restart an interface without having to restart a whole server.
- Once you make changes to a network interface configuration, you need to force that interface to reread its configuration file. You accomplish this by bringing the interface down with the ifdown command and then bringing it back up with ifup command.
- When the ifup command is executed, the configuration file is reread and the interface is brought back into operation with its newly minted parameters.
- The `/etc/network/interfaces` file is used to tell ifup and ifdown how to configure network interfaces as those interfaces are brought up and down.

---

## Network Interface Bonding
- There are times when you need more bandwidth than a single interface can provide, or you want some form of link redundancy in case of a cabling or other network problem.
- This link redundancy function goes by many names, depending on the vendor: EtherChannel, VMwarePortGroups, Bonds, and Link Aggregation Groups (LAG) are just a few.
- Linux also provides this capability and calls it bonding. It allows you to create a single logical network link that comprises multiple physical links and that scales up as you add more interfaces, can provide load balancing across the interfaces, and can provide failover protection.
- To use network bonding, you need to install the bonding kernel module via the `modprobe` command. Modprobe allows you to add the additional capability to the Linux Kernal. It works like this:
  - `sudo modprobe bonding`
- Let's create a bond which will include eth1 and eth2 interfaces, we must take the interfaces down before adding them to the bond.

    ```
    vagrant@ubuntulvmlab0101:~$ sudo ip link show
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
        link/ether 08:00:27:d9:fa:93 brd ff:ff:ff:ff:ff:ff
        altname enp0s8
    6: eth2: <BROADCAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
        link/ether aa:1a:18:7a:36:de brd ff:ff:ff:ff:ff:ff
    7: eth1: <BROADCAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
        link/ether 3a:27:9d:31:51:8c brd ff:ff:ff:ff:ff:ff
        
    vagrant@ubuntulvmlab0101:~$ sudo ip link add bond0 type bond mode 802.3ad
    vagrant@ubuntulvmlab0101:~$ sudo ip link set eth1 master bond0
    vagrant@ubuntulvmlab0101:~$ sudo ip link set eth2 master bond0
    
    vagrant@ubuntulvmlab0101:~$ sudo ip link set bond0 up
    vagrant@ubuntulvmlab0101:~$ ip link show bond0
    8: bond0: <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
        link/ether fa:ff:12:fe:50:df brd ff:ff:ff:ff:ff:ff
        
    vagrant@ubuntulvmlab0101:~$ cat /proc/net/bonding/bond0
    Ethernet Channel Bonding Driver: v5.15.0-160-generic
    
    Bonding Mode: IEEE 802.3ad Dynamic link aggregation
    Transmit Hash Policy: layer2 (0)
    MII Status: up
    MII Polling Interval (ms): 100
    Up Delay (ms): 0
    Down Delay (ms): 0
    Peer Notification Delay (ms): 0
    
    802.3ad info
    LACP active: on
    LACP rate: slow
    Min links: 0
    Aggregator selection policy (ad_select): stable
    
    Slave Interface: eth1
    MII Status: up
    Speed: Unknown
    Duplex: Unknown
    Link Failure Count: 0
    Permanent HW addr: 3a:27:9d:31:51:8c
    Slave queue ID: 0
    Aggregator ID: 1
    Actor Churn State: monitoring
    Partner Churn State: monitoring
    Actor Churned Count: 0
    Partner Churned Count: 0
    
    Slave Interface: eth2
    MII Status: up
    Speed: Unknown
    Duplex: Unknown
    Link Failure Count: 0
    Permanent HW addr: aa:1a:18:7a:36:de
    Slave queue ID: 0
    Aggregator ID: 2
    Actor Churn State: monitoring
    Partner Churn State: monitoring
    Actor Churned Count: 0
    Partner Churned Count: 0
    ```
- Bonding when incorrectly configured or cabled, can be the source of some pretty messy network problems; 802.3ad runs the Link Aggregation Protocol (LACP) with the switch or server at the other end of the bond to make sure that everything is connected correctly.

### Linux network bonding modes
- Linux currently support the following bonding modes:

| Bonding Mode Type | Description                                                                                                                                                                |
| ----------------- |----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **balance-rr** | The default, round-robin bonding, which provides loiad balancing and fault tolerance                                                                                       | 
| **active-backup** | Provides fault tolerance whereby only one slave can be active at a time and, it it fails, the other slave takes over                                                       |
| **balance-xor** | Provides fault tolerance and load balancing by transmitting based on hash                                                                                                  |
| **broadcast** | Provides fault tolerance by transmitting everything on all slave interfaces                                                                                                |
| **802.3ad IEEE** | Standard for dynamic link aggregation creates aggregation groups for links that share the same speed and duples in order to provide for fault tolerance and loadbalancing. |
| **balance-tlb** | Adaptive transmit load balancing that doesn't require any special switch support                                                                                           |
| **balance-alb** | Adaptive load balancing that doesn't require any special switch support due to its use of ARP negotiation                                                                  |