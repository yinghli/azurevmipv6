# Create a dual stack IPv4/IPv6 VNET with 2 VMs

This template demonstrates creation of a dual stack IPv4/IPv6 VNET with 2 dual stack VMs and a STANDARD Load Balancer.

The template creates the following Azure resources:

- a dual stack IP4/IPv6 Virtual Network (VNET) with a dual stack subnet
- a virtual network interface (NIC) for each VM with both IPv4 and IPv6 endpoints
- an Internet-facing STANDARD Load Balancer with an IPv4 and an IPv6 Public IP addresses
- IPv6 Network Security Group rules (allow HTTP and RDP)
- an IPv6 User-Defined Route to a fictitious Network Virtual Appliance
- an IPv4 Public IP address for each VM to facilitate remote connection to the VM (RDP)
- two virtual machines with both IPv4 and IPv6 endpoints in the VNET/subnet

There are two json template, [one](https://github.com/yinghli/azurevmipv6/blob/master/ipv6.json) is for Windows Server 2019, the [other](https://github.com/yinghli/azurevmipv6/blob/master/ipv6ubuntu.json) is Ubuntu 18.04. </br>
Go to Azure portal, follow [this](https://docs.microsoft.com/en-us/azure/virtual-network/ipv6-configure-standard-load-balancer-template-json) to deploy it.</br>

## Windows 
For Windows Server, the VM can boot automatically. Login the system, ipconfig will show the IPv4/IPv6 configuration. </br>
- Install IIS and make sure website are working well. 
-	Turn off windows firewall Or make sure TCP 80 are open for connections.
-	Open browser on client, type “http://[ipv6 address]” to access the testing website. 
- Open CMD and “netstat -an” to check connections are all good. 

## Ubuntu
For Ubuntu Server, system can't boot up and get below errors:
```
[   15.785613] cloud-init[948]: Cloud-init v. 19.2-36-g059d049c-0ubuntu2~18.04.1 running 'init' at Wed, 30 Oct 2019 08:03:54 +0000. Up 15.27 seconds.
[   15.795612] cloud-init[948]: ci-info: +++++++++++++++++++++++++++Net device info++++++++++++++++++++++++++++
[   15.800464] cloud-init[948]: ci-info: +--------+-------+-----------+-----------+-------+-------------------+
[   15.804712] cloud-init[948]: ci-info: | Device |   Up  |  Address  |    Mask   | Scope |     Hw-Address    |
[   15.808940] cloud-init[948]: ci-info: +--------+-------+-----------+-----------+-------+-------------------+
[   15.814162] cloud-init[948]: ci-info: |  eth0  | False |     .     |     .     |   .   | 00:0d:3a:f9:b7:61 |
[   15.818731] cloud-init[948]: ci-info: |   lo   |  True | 127.0.0.1 | 255.0.0.0 |  host |         .         |
[   15.823138] cloud-init[948]: ci-info: |   lo   |  True |  ::1/128  |     .     |  host |         .         |
[   15.827898] cloud-init[948]: ci-info: +--------+-------+-----------+-----------+-------+-------------------+
[   15.832310] cloud-init[948]: ci-info: +++++++++++++++++++Route IPv6 info+++++++++++++++++++
[   15.836468] cloud-init[948]: ci-info: +-------+-------------+---------+-----------+-------+
[   15.842615] cloud-init[948]: ci-info: | Route | Destination | Gateway | Interface | Flags |
[   15.848192] cloud-init[948]: ci-info: +-------+-------------+---------+-----------+-------+
[   15.853527] cloud-init[948]: ci-info: +-------+-------------+---------+-----------+-------+
[   15.858639] cloud-init[948]: 2019-10-30 08:03:55,402 - azure.py[ERROR]: Failed to read /var/lib/dhcp/dhclient.eth0.leases: [Errno 2] No such file or directory: '/var/lib/dhcp/dhclient.eth0.leases'
[   15.867224] cloud-init[948]: 2019-10-30 08:03:55,404 - azure.py[WARNING]: No lease found; using default endpoint
```

Need to login the server serial console from portal and run dhclient command. 

```
dhclient -4
dhclient -6
```
Simplely install apache and php for testing. 
```
sudo apt-get update 
sudo apt-get install apache2
sudo apt-get install php libapache2-mod-php php-mysql
```
Create new info.php page to capture HTTP header information for each request.</br>
```
vi /var/www/html/info.php

<?php
// Show all information, defaults to INFO_ALL
phpinfo();
?>
```
Client can initial HTTP request to standard load balancer IPv6 address. </br>
Here is "netstat -anW" output. "W" is used to wide output. 
```
root@DsVM1:/var/www/html# netstat -anW
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address                   State
tcp        0      0 127.0.0.53:53           0.0.0.0:*                         LISTEN
tcp        0      0 0.0.0.0:22              0.0.0.0:*                         LISTEN
tcp        0    256 10.0.0.5:22             167.220.233.144:53644             ESTABLISHED
tcp6       0      0 :::80                   :::*                              LISTEN
tcp6       0      0 :::22                   :::*                              LISTEN
tcp6       0      0 ace:cab:deca:deed::5:80 fe80::1234:5678:9abc:61714        SYN_RECV
tcp6       0      0 10.0.0.5:80             168.63.129.16:61706               SYN_RECV
tcp6       0      0 ace:cab:deca:deed::5:80 2404:f801:8050:1:80c0::9359:1620  FIN_WAIT2
udp        0      0 127.0.0.53:53           0.0.0.0:*
udp        0      0 0.0.0.0:68              0.0.0.0:*
udp6       0      0 fe80::20d:3aff:fef9:b761:546 :::*
```

Follow [this](https://docs.microsoft.com/en-us/azure/network-watcher/network-watcher-nsg-flow-logging-portal) to enable NSG flow log. Download the nsg flow log from storage account, will get the HTTP request log like this:

```
"1572419632,168.63.129.16,10.0.0.5,56372,80,T,I,A,B,,,,",
"1572419632,fe80::1234:5678:9abc,ace:cab:deca:deed::5,56383,80,T,I,A,B,,,,",
"1572419634,2404:f801:8050:1:80c0::9359,ace:cab:deca:deed::5,30278,80,T,I,A,E,14,1557,13,24562",
```

First two logs are probe message from standard load balancer(SLB). </br>
`169.63.129.16` and `fe80::1234:5678:9abc` are SLB probe packet source IP. </br>
"T" means TCP, "I" is inbound direction. "A" is traffic is allowed. "B" is initial packet. </br>

Third one the customer real IPv6 request.</br>
`2404:f801:8050:1:80c0::9359` is client source IPv6 address. `ace:cab:deca:deed::5` is server IPv6 address. </br>
30278 is source port, 80 is destination port. </br>
"T" means TCP, "I" is inbound direction. "A" is traffic is allowed. "E" means connection closed. </br>
14 packets client sent to server, 1557 bytes. 13 packets server send to client, 2456 bytes. </br>

For more flog log information, check [here](https://docs.microsoft.com/en-us/azure/network-watcher/network-watcher-nsg-flow-logging-overview#nsg-flow-logs-version-2)
