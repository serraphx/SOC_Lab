Lets begin by with spinning up the OPNsense firewall and apply some basic configurations to get the environment started.

One thing that has to be taken into account when configuring all these VMs is the resource allocation. For the system that will be used to host the lab, the limiting factor will be the CPU. For the CPU in use, it utilizes both P-Cores and E-Cores, and we will be mapping the core usage using core affinity. Both VirtualBox and Docker Compose allow for this configuration, which will ensure appropriate resource allocation to each of the different servers/services in use.

This machine will be configured with 1 P-core (0), 1 E-core (8)4 GB	RAM, 8 GB HD, 2 NICs: 1 set to NAT, 1 set to internal network (Admin_VLAN). We will be expanding the number of internal networks as we begin to build the different subnets. It is important to note that you will need to make sure that OS type is set to BSD and version is FreeBSD 64-bit as VirtualBox will default to unknown, which will prevent the machine from properly booting.

Once the installation process has been completed, the root password was changed as leaving default passwords is bad practice. In order to be able to continue configuration utilizing the GUI, the WAN and the LAN interfaces need to be set. 
 