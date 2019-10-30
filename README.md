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

There are two json template, one is for Windows Server 2019, the other is Ubuntu 18.04. </br>
For Windows Server, the VM can boot automatically. Login the system, ipconfig will show the IPv4/IPv6 configuration. </br>
For Ubuntu Server, system boot up and get below errors:
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

Need to login the server console and run dhclient command. 

```
dhclient -4
dhclient -6
```
