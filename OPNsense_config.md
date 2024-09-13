Lets begin by with spinning up the OPNsense firewall and apply some basic configurations to get the environment started.

One thing that has to be taken into account when configuring all these VMs is the resource allocation. For the system that will be used to host the lab, the limiting factor will be the CPU. For the CPU in use, it utilizes both P-Cores and E-Cores, and we will be mapping the core usage using core affinity. Both VirtualBox and Docker Compose allow for this configuration, which will ensure appropriate resource allocation to each of the different servers/services in use.

This machine will be configured with 1 P-core (0), 1 E-core (8)4 GB	RAM, 8 GB HD, 2 NICs: 1 set to NAT Network (named Lab_NAT_Network), 1 set to internal network (Admin_VLAN). We will be expanding the number of internal networks as we begin to build the different subnets. It is important to note that you will need to make sure that OS type is set to BSD and version is FreeBSD 64-bit as VirtualBox will default to unknown, which will prevent the machine from properly booting. To set the core affinity, the VBoxManage command needs to be utilized. First, check to make sure the the VM is listed with the command `VBoxManage list vms`. Once your OPNsense box has been indentified, you then need to make sure the correct core count is set, you can use the command `VBoxManage modifyvm "OPNsense" --cpus 2`. Finally, the command to set the appropriate affinity is `VBoxManage setextradata "OPNsense" "VBoxInternal/CPUM/HostCPUID0/CPUAffinity" "0,8"`.

<!-- photo[OPNsense cpu affinity] -->
<img src="https://i.imgur.com/sQGsTQK.png"/>



Once the installation process has been completed, the root password was changed as leaving default passwords is bad practice. In order to be able to continue configuration utilizing the GUI, the WAN and the LAN interfaces need to be set. For the WAN, it will be left at the default which is to use DHCP, much as it would be when connecting with an ISP. For the LAN, a static IP was set 10.0.1.10, which is the assigned IP for the OPNsense firewall. To test the connectivity, I pinged 8.8.4.4, receiving all packets, confirming that the server is connected and receiving packets through the NAT Network.

<!-- photo[Opnsense_connect_confirm] -->
<img src="https://i.imgur.com/TI2ST16.png"/>

Utilizing a Kali box, I will continue the set up for OPNsense, though this can also be accomplished utilizing a CLI. What is important to note that when you first connect your Kali box to the internal network, you will need to the IP address as there is not DHCP on the internal network of VirtualBox unlike when you use NAT or NAT Network modes. For this, I made modifications to the Kali box's network interface configuration file `/etc/network/interfaces`.

```
# NAT Network interface
auto eth0
iface eth0 inet dhcp

# Internal Network interface
auto eth1
iface eth1 inet static
address 10.0.1.100
netmask 255.255.255.0
```

<!-- photo[Kali box network config]-->
<img src="https://i.imgur.com/B3R015Z.png"/>

Once the Kali box was configured, both the OPNsense firewall and Kali box are able to ping one another using the internal network (10.0.1.0/24 subnet).

<!-- photo[Kali-opnsense ping connect] -->
<img src="https://i.imgur.com/QwF42GQ.png"/>

The first thing that I did once I was able to log into the web GUI for the OPNsense firewall was to run the updater to ensure that the firewall was up to date. 

Next we will begin to create each of the VLANs that will be used for the network. To create the VLANs, in the GUI, you will navigate to Interfaces -> Other Types -> VLAN. For each of the VLANs, when creating them, the device name will be vlan0. plus the subnet address (ie vlan0.10.0.10.0 for Admin_VLAN), Parent will be set to the LAN adapter, the VLAN tag in occardance to the network map, VLAN priority set to default, and finally the appropriate description of the VLANs.

<!-- photo[VLAN config] -->
<img src="https://i.imgur.com/b8E75Dm.png"/>

| VLAN Name  | VLAN ID | Subnet         |
|------------|---------|----------------|
| Admin_VLAN | 10      | 10.0.10.0/24    |
| SOC_VLAN   | 20      | 10.0.20.0/24    |
| DMZ_VLAN   | 30      | 10.0.30.0/24    |
| Lab_VLAN   | 40      | 10.0.40.0/24    |
| RT_VLAN    | 50      | 10.0.50.0/24    |

<!-- photo[VLAN list] -->
<img src="https://i.imgur.com/iOUtUXd.png"/>

The next steps will be to write the firewall rules for each of these VLANs that take into account the needs of each of the services that will be in each VLAN and the appropriate traffic to and from those services.

10.0.10.0 rules