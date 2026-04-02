OPNsense has the powerful feature of setting up a redundant firewall with automatic fail-over option. This feature is referred to as **High Availability** and is made possible by the **Common Address Redundancy Protocol (CARP)** which allows multiple hosts to share the same IP address and Virtual Host ID (VHID) in order to provide *high availability* for one or more services.

This means that one or more hosts can fail, and the other hosts will transparently take over so that users do not see a service failure. However, all fail-safe interfaces should have a dedicated IP address which will be combined with one shared virtual IP address to communicate to both networks.

**CARP**

Common Address Redundancy Protocol uses IP protocol 112, is derived from OpenBSD and uses multicast packets to signal its neighbours about its status. We must ensure that each interface can receive CARP packets. Every virtual interface must have a unique Virtual Host ID (vhid), which is shared across the physical machines. To determine which physical machine has a higher priority, the advertised skew is used. A lower skew means a higher score. (our master firewall uses 0).

**pfSync**

Together with CARP, we can use pfSync to replicate our firewalls state. When failing over we need to make sure both machines know about all connections to make the migration seamless. It’s highly advisable to use a dedicated interface for pfSync packets between the hosts, both for security reasons (state injection) as well as for performance.

When using different network drivers on both machines, like running a HA setup with one physical machine as master and a virtual machine as slave, states can not be synced as interface names differ. The only workaround would be to set up a LAGG.

**XMLRPC sync**

OPNsense includes a mechanism to keep the configuration of the backup server in sync with the master. This mechanism is called XMLRPC sync and can be found under `System`  ‣ `High Availability`  ‣ `Settings` .

Cloning our OPNsense VM.

Since we are using a virtualized instance of OPNsense, the easiest way forward for purposes of this lab would be simply clone our instance. In Virtual Machine Manager, right click the instance and click `Clone`

![alt text](./resources/973e24404c404d9b81ad9f81fbad57e3.png)

Ensure to keep the first option under the `Storage` tab check so that new storage and MAC address are created for the clone

![alt text](./resources/ecb658dc4e21468ebf20698ae39061d1.png)


Once created, we can go ahead and rename them appropriately in the `Overview` tab. We can also observe two unique NICs on each VM.

![alt text](./resources/0638246192a94f1fa92bc66d615ad259.png)


Networking the Master and Backup firewall

We need to create an additional NAT network between our 2 firewalls since CARP and pfSync require a shared layer 2 environment where they can exchange multicast packets for direct communication. In "Routed" mode, our Host machine acts as a router which doesn't handle traffic leading to our VMs not having internet access.

In Virtual Machine Manager, go to `Edit` ‣ `Connection details` and click the `+` button at the bottom to create a new network. Assign the network address and enable DHCP so that it can supply addresses to the WAN interfaces of the firewalls

![alt text](./resources/0977f52b90f34c8f8934f64dccf0ac8a.png)


Next we need to connect our firewalls to the NAT network. First we need to ensure that each firewall instance is connected to the NAT network. In Virtual Machine Manager, right click the Master firewall instance and click `Open` then click the lightbulb icon. Change the NIC connection attached to QEMU/KVM's default "NAT mode" and attach it to our new NAT network.

![alt text](./resources/5991ab2a13ef43d1bbfc174ddd54f711.png)

![alt text](./resources/cc97d1e1b1cd4a5994d8eed99744d8fd.png)


Next add a new NIC to our Master firewall and attach it to our LAN network. Take note of the MAC address and naming schemes in QEMU/KVM are not enabled by default.

![alt text](./resources/5a0414acee3e4b33880c0f58bdb7e74a.png)


Repeat the above steps for our backup firewall as well.

Next we start up the backup firewall first to avoid IP addressing conflicts with the master firewall. Initially it will boot up with the same address as our Master so we need to change it. To do this, login into the GUI, navigate to `Interfaces` ‣ `[LAN]`  and make sure it's set to "Static IPv4" and assign it a new IPv4 address. Click `Save` and apply changes

![alt text](./resources/217b8c53c43d4c6abcb663f2ef1c02d4.png)


You might have to reboot the backup firewall and it should boot up with the newly assigned IP address

![alt text](./resources/98054b2b59544b1fa2f25cb4e44fc3ae.png)


Go ahead and boot up the master firewall instance. Login to the GUI go to `Interfaces`  ‣ `Assignments` and add a description for the interface attached to the third NIC we created earlier and click `Add`. Do this on both firewalls using the GUI

![alt text](./resources/57e6175822f048d880e37d3412dcfa93.png)


Next go to `Interfaces`  ‣ `[pfsync]` and check `Enable Interface`  and assign it a a completely separate, dedicated subnet that is not used by the LAN or WAN. In this case I used the 10.0.2.0/24 subnet with 10.0.2.1 as my Master and 10.0.2.2 as my backup. Click `Save` and Apply changes.

![alt text](./resources/afc51642023b4c989c33400fd3e1e7ff.png)


Next we need to make sure the appropriate protocols can be used on the different interfaces, go to `Firewall`  ‣ `Rules` ‣ `pfsync` and make sure both LAN and WAN accept at least CARP packets. Because we’re connecting both firewalls using a direct cable connection, we will add a single rule to accept all traffic on all protocols for that specific interface. Click `Save` and Apply changes. We need to create this rule on both firewalls.

![alt text](./resources/4a7113f244c649e782f956e238da3673.png)


Next, on the master node we are going to setup our Virtual IP addresses, which will also be added to the backup node with a higher skew after synchronisation. Go to `Interfaces`  ‣ `Virtual IPs`  and add a new one with the following settings

|     |     |
| --- | --- |
| Type | Carp |
| Interface | WAN |
| IP addresses | 10.0.1.254 / 24 |
| Virtual password | *\[sabula1\]* |
| VHID Group | 1   |
| Advertising Frequency | Base 1 / Skew 0 |
| Description | Virtual IP WAN |

|     |     |
| --- | --- |
| Type | Carp |
| Interface | LAN |
| IP addresses | 10.0.0.254 / 24 |
| Virtual password | opnsense (the example uses this) |
| VHID Group | 3   |
| Advertising Frequency | Base 1 / Skew 0 |
| Description | Virtual IP LAN |

It is good practice to create Carp Virtual IPs with the same subnet mask as its parent interface. If the parent interface is `/24`, your Carp VIP should also be `/24`.

![alt text](./resources/3bf65efe2eae4f1ebc124235289c7b0c.png)


We need to replicate the same setup on our backup firewall this time with a higher skew of 100

![alt text](./resources/4f144f6ac4dd4ae18597e3a491002fee.png)


We also need to setup outbound NAT because traffic going out of the firewall should also use the virtual IP address on the WAN interface to make seamless transitions possible.The default NAT configuration is for OPNsense is to use Automatic outbound NAT rule generation using the WAN interface’s IP address for outgoing connections which will not allow seamless transitions unless changed to the WAN Virtual IP.

Go to `Firewall`  ‣ `NAT`  ‣ `Outbound`. Choose manual outbound nat rule generation.

![alt text](./resources/b6cb6d503a804c44834784601b7706e3.png)


On this page create the a rule originating from the 10.0.0.0/24 network (LAN Network) to use the CARP virtual interface (10.0.1.254/24). The rule should contain the following:

|     |     |
| --- | --- |
| Interface | WAN |
| Source address | LAN net (10.0.0.0/24) |
| Translation / target | 10.0.1.254/24 (CARP virtual IP WAN) |

![alt text](./resources/a2a6607cd6de4bf597b4a08c63af64ff.png)

![alt text](./resources/19e2622496b0452fa76c4b72f2c97535.png)


Next we need to setup pfSync and HA sync (xmlrpc) by first configuring pfSync to synchronize the connection state tables and HA sync (xmlrpc) on the master firewall. Go to `System`  ‣ `High Availability`  ‣ `Settings`  and enable pfSync by selecting pfsync from the Synchronize all states via dropdown and enter the peer IP (10.0.2.2) in the field Synchronize Peer IP.

![alt text](./resources/5bd82295dc70488daf0b5417f25a33c2.png)


To synchronize the configuration settings from the master to the backup firewall, we setup the XMLRPC sync. In the Synchronize Config to IP field we enter the peer IP (10.0.2.2) of the pfsync interface again to keep this traffic on the direct connection between the two firewalls. Now we need to enter the remote user name and password and configure the services we want to duplicate to the backup server(I have selected all for the sake of the lab). Click `Save`

![alt text](./resources/d1baba833a8b4f69a740ead9963c76d9.png)


After this we configure pfSync on the backup firewall. Go to `System`  ‣ `High Availability`  ‣ `Settings`  and enable pfSync by activating the Synchronize States checkbox, selecting pfsync for the Synchronize Interface and enter the master IP (10.0.2.1) in the field Synchronize Peer IP. Do not configure XMLRPC sync on the backup firewall.

![alt text](./resources/e98b64ff4c86464aa5a69e516e15132a.png)


Just to make sure all settings are properly applied, reboot both firewalls before testing. Then go to `System`  ‣ `High availability`  ‣ `Status`  in the OPNsense GUI and check if both machines are properly initialized. (It is normal that the "Backup firewall is not accessible" error only appears on the **Backup's** GUI since the Backup is not configured to reach back and sync the Master

![alt text](./resources/4b328d564776445fa591b6b80fe4428c.png)


To test our setup, we can simulate hardware failure by simply shutting down our Master firewall. Once it is off, in the backup OPNsense GUI go to `Interfaces`  ‣ `VIP` ‣ `Status` . It should now say MASTER.

![alt text](./resources/61faf4b6383d451a884343acdb5e0ab8.png)
