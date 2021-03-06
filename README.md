# DEVNET-2062 Workshop "Application Engineered Egress Routing"

## Introduction ##
This documentation provides hands on examples of the SR Host Networking demonstration provided during the workshop. The following tasks will be performed:
* Whitelisting(capturing) of a Linux interface by VPP (Vector Packet Processing)
* View the interface details via the VPP shell/cli.
* Execute a Python code snippet to display the VPP interface information in a programmatic fashion.

## Setup ##

Password for Mac Machines is cisco123

Change dir to ~/labuser/devnet-rekha

A Vagrant Ubuntu 16.04 (Xenial) box setup having VPP and associated packages pre-installed is running on your workbench. 

Login into the Ubuntu VM (srhost.box) using the following command:

<pre>
<b>vagrant ssh</b>

<i>Welcome to Ubuntu 16.04.1 LTS (GNU/Linux 4.4.0-47-generic x86_64)

&ltsnip&gt

vagrant@ubuntu-xenial:~$</i>

</pre>

## Steps ##

### Whitelisting Linux interface via VPP ###



1. Execute the following command to see what relevant packages are installed on the box:

<pre>
<b>workshop/show-pkgs</b>

<i>Current working directory:
/home/vagrant


Ubuntu Release Version:
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 16.04.1 LTS
Release:	<b>16.04</b>
Codename:	xenial


Vpp and Docker Package Info:
ii  vpp                              17.04-release                              amd64        Vector Packet Processing--executables
ii  vpp-api-python                   17.04-release                              amd64        VPP Python API bindings
ii  vpp-dpdk-dkms                    17.01.1-release                            amd64        DPDK 2.1 igb_uio_driver
ii  vpp-lib                          17.04-release                              amd64        Vector Packet Processing--runtime libraries
ii  vpp-plugins                      17.04-release                              amd64        Vector Packet Processing--runtime plugins
ii  docker-ce                        17.03.1~ce-0~ubuntu-xenial                 amd64        Docker: the open-source application container engine
</i>
</pre>



2. Display the VPP version (vppctl is the vpp shell). 

<pre>
<b>sudo vppctl show version </b>

<i>vpp v17.04-release built by jenkins on ubuntu1604-basebuild-4c-4g-2454 at Fri Apr 21 15:57:33 UTC 2017
</i>
</pre>


3. List the interfaces on the SRHost box machine. (Examine the interface <b>enp0s8</b> - its state is UP)

<pre>
<b>ifconfig</b>


<i>docker0   Link encap:Ethernet  HWaddr 02:42:8a:f8:a9:6a
          inet addr:172.17.0.1  Bcast:0.0.0.0  Mask:255.255.0.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

enp0s3    Link encap:Ethernet  HWaddr 02:8c:e2:cf:b5:33
          inet addr:10.0.2.15  Bcast:10.0.2.255  Mask:255.255.255.0
          inet6 addr: fe80::8c:e2ff:fecf:b533/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:886 errors:0 dropped:0 overruns:0 frame:0
          TX packets:671 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:86672 (86.6 KB)  TX bytes:88364 (88.3 KB)

<b>enp0s8</b>    Link encap:Ethernet  HWaddr 08:00:27:f3:59:d7
          inet addr:172.28.128.4  Bcast:172.28.128.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:fef3:59d7/64 Scope:Link
          <b>UP</b> BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:20 errors:0 dropped:0 overruns:0 frame:0
          TX packets:21 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:4424 (4.4 KB)  TX bytes:2996 (2.9 KB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

</i>
</pre>

4. Note the PCI bus information associated with the interface <b>enp0s8</b>

<pre>

<b>sudo lshw -class network -businfo </b>

<i>
Bus info          Device     Class       Description
====================================================
pci@0000:00:03.0  enp0s3     network     82540EM Gigabit Ethernet Controller
pci@<b>0000:00:08.0</b>  enp0s8     network     82540EM Gigabit Ethernet Controller

</i>
</pre>

5. Change current dir to /etc/vpp. Then execute the following command to create the correct startup configuration file:

<pre>

<b>cd /etc/vpp 

sudo cp startup.conf.demo startup.conf </b>

</pre>

The above command changes the dpdk section of VPP's startup configuration file to contain 'dev' entries for the PCI bus information obtained in the previous step. It placed the following configuration in /etc/vpp/startup.conf file.
<pre>
<i>.....
&ltsnip&gt
dpdk {

	## Whitelist specific interface by specifying PCI address
	 dev  0000:00:08.0


	## Change UIO driver used by VPP, Options are: igb_uio, vfio-pci
	## and uio_pci_generic (default)
	 uio-driver igb_uio
 }
 </i>
</pre>


6. Set interface state for <b> enp0s8 </b> to DOWN and flush it

<pre>
<b>sudo ifconfig enp0s8 down
sudo ip addr flush dev enp0s8

ip link show</b>
<i>
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 02:8c:e2:cf:b5:33 brd ff:ff:ff:ff:ff:ff
    3: <b>enp0s8</b>: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast <b>state DOWN</b> mode DEFAULT group default qlen 1000
    link/ether 08:00:27:45:90:2c brd ff:ff:ff:ff:ff:ff
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default
    link/ether 02:42:29:e0:b4:14 brd ff:ff:ff:ff:ff:ff
    </i>
</pre>

7. Restart the VPP service

<pre>

<b>sudo service vpp restart

sudo service vpp status
</b>
</pre>

8. Assign and IP and bring the interface state to up

<pre>
<b>sudo vppctl set interface ip address GigabitEthernet0/8/0 172.28.128.3/24

sudo vppctl set interface state GigabitEthernet0/8/0 up</b>

</pre>


### VPP Shell/cli ###


1. List the VPP interfaces using the VPP shell/cli

<pre>
<b>sudo vppctl show interface </b>

<i>
              Name               Idx       State          Counter          Count
GigabitEthernet0/8/0              1         up
local0                            0        down</i>


<b>sudo vppctl show interface address</b>
<i>
GigabitEthernet0/8/0 (up):
  172.28.128.3/24
local0 (dn):
</i>
</pre>

2.Exit the VM (srhost.box) and ping the interface from outside.

<pre>
vagrant@ubuntu-xenial:/etc/vpp$ <b>exit</b>
<i>logout
Connection to 127.0.0.1 closed.



$</i> <b>ping 172.28.128.3 </b>
<i>
PING 172.28.128.3 (172.28.128.3): 56 data bytes
64 bytes from 172.28.128.3: icmp_seq=0 ttl=64 time=0.465 ms
64 bytes from 172.28.128.3: icmp_seq=1 ttl=64 time=0.164 ms
^C
--- 172.28.128.3 ping statistics ---
2 packets transmitted, 2 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 0.164/0.315/0.465/0.151 ms

</pre>

3. Examine the ip arp table in VPP

<pre>
$</i> <b>vagrant ssh </b>

<i>Welcome to Ubuntu 16.04.1 LTS (GNU/Linux 4.4.0-47-generic x86_64)

&ltsnip&gt
vagrant@ubuntu-xenial:~$
vagrant@ubuntu-xenial:~$</i>

<b>
sudo vppctl show ip arp </b>
<i>
    Time           IP4       Flags      Ethernet              Interface
    219.3631  172.28.128.1    DN    0a:00:27:00:00:00   GigabitEthernet0/8/0
</i>
</pre>



### Python - PAPI Interface in VPP ###



1. List the VM's interfaces using the vppctl shell/cli


<pre>

<b>sudo vppctl show interface</b>
<i>

              Name               Idx       State          Counter          Count
GigabitEthernet0/8/0              1         up
host-vpp1                         2         up
local0                            0        down

</i>
</pre>



2. Execute Python script to List the interfaces using VPP's Python API (PAPI)

<pre>


<b> sudo workshop/test-papi.py </b>

<i>
 Connecting to VPP instance




 VPP show version:
 17.04-release




The interfaces in the VPP instance are:
local0
GigabitEthernet0/8/0
host-vpp1



 Disconnecting from VPP instance
 
 </i>
 </pre>
 

### Optional Exercise: ###

#### Create veth pairing between VPP and Docker ####

1. Create a Linux veth pair. 

<pre>
<b>sudo ip link add name veth_vpp1 type veth peer name vpp1
ip link show </b>
<i>
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 02:8c:e2:cf:b5:33 brd ff:ff:ff:ff:ff:ff
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default
    link/ether 02:42:cc:24:e8:90 brd ff:ff:ff:ff:ff:ff
5: vpp1@veth_vpp1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether ce:18:0a:5a:ef:4f brd ff:ff:ff:ff:ff:ff
6: veth_vpp1@vpp1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 7e:b0:de:56:a1:3c brd ff:ff:ff:ff:ff:ff
</i>
</pre>


2. Create a host pair that will attach to a Linux AF_PACKET interface, one side of the veth pair.

<pre>
<b>sudo vppctl create host-interface name vpp1 </b>
<i>
host-vpp1 </i>

</pre>

3. Give an ip address to the interface and change its state to up

<pre>
<b>
sudo vppctl set interface state  host-vpp1 up
sudo vppctl set interface ip address host-vpp1 172.16.1.1/24
sudo vppctl show int  address</b>
<i>
GigabitEthernet0/8/0 (up):
  172.28.128.3/24
host-vpp1 (up):
  172.16.1.1/24
local0 (dn):
</i>
</pre>


4. List docker images

<pre>
<b>sudo docker images</b>
<i>
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
alpine              latest              665ffb03bfae        7 days ago          3.97 MB
</i>
</pre>


5. Create docker network to plumb docker side to the veth pair

<pre>
<b>sudo docker network create -d macvlan --subnet=172.16.1.0/24 --gateway=172.16.1.1 -o parent=veth_vpp1 vpp1_net

sudo docker network ls  </b>
<i>
NETWORK ID          NAME                DRIVER              SCOPE
c1ca29db93b5        bridge              bridge              local
4a195de3a0e4        host                host                local
c1ab80322571        none                null                local
91d993c13757        vpp1_net            macvlan             local
</i>
</pre>

6. Launch the Docker Container and show its “ip address” from inside the container.

<pre>
<b>sudo docker rm -f $(sudo docker ps -a -q)

sudo docker run --name=guest_container --net=vpp1_net --ip=172.16.1.101 -it alpine /bin/sh </b>
<i>
/ # ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
8: eth0@if6: <NO-CARRIER,BROADCAST,MULTICAST,UP,M-DOWN> mtu 1500 qdisc noqueue state LOWERLAYERDOWN
    link/ether 02:42:ac:10:01:65 brd ff:ff:ff:ff:ff:ff
    inet 172.16.1.101/24 scope global eth0
       valid_lft forever preferred_lft forever
       
</i>
</pre>

### GitHub Repository Information ###

GitHub Repository:  https://github.com/cisco-ie/sr-host-networking

Contact Information: Rekha Rawat 

Email: rrawat@cisco.com


### Useful Links ###

FD.io VPP Wiki: https://wiki.fd.io/view/VPP

VPP Python API: https://wiki.fd.io/view/VPP/Python_API

GoBGP: https://github.com/osrg/gobgp  









