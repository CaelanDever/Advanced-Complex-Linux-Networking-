# Advanced-Complex-Linux-Networking

# Description:

Configure advanced networking features and services on a CentOS server to optimize network performance, security, and reliability.

Step-by-Step Guide

# 1. Implement VLANs

1.1 Create VLAN Interface
To assign VLAN ID 10 to the ens192 interface, create a VLAN sub-interface using the ip command:

ip link add link ens192 name ens192.10 type vlan id 10

1.2 Configure IP Address for VLAN Interface

Assign an IP address to the VLAN interface ens192.10:

ip addr add 10.0.0.98/24 dev ens192.10

1.3 Persist Configuration (Optional)

To make these configurations persistent across reboots, create a configuration file /etc/sysconfig/network-scripts/ifcfg-ens192.10 and add the following content:

DEVICE=ens192.10
BOOTPROTO=static
ONBOOT=yes
IPADDR=10.0.0.98
PREFIX=24
VLAN=yes

Then, restart the network service:

systemctl restart network

# 2. Set Up Bonding/Teaming

2.1 Configure Bonding Interface

Edit the bonding interface configuration file /etc/sysconfig/network-scripts/ifcfg-bond0:

DEVICE=bond0
TYPE=Bond
BONDING_MASTER=yes
BOOTPROTO=none
ONBOOT=yes
BONDING_OPTS="mode=1 miimon=100"

2.2 Configure Slave Interface (ens192)

Edit the network configuration for ens192 to make it a slave interface of the bonding interface:

vi /etc/sysconfig/network-scripts/ifcfg-ens192
Add the following content:

DEVICE=ens192
TYPE=Ethernet
BOOTPROTO=none
ONBOOT=yes
MASTER=bond0
SLAVE=yes

2.3 Restart the Network

Once configured, restart the network service:

systemctl restart network

# 3. Implement IP Routing

3.1 Enable IP Forwarding

Edit the /etc/sysctl.conf file to enable IP forwarding:

vi /etc/sysctl.conf

Uncomment or add the following line:

net.ipv4.ip_forward = 1

Apply the changes:

sysctl -p

3.2 Add Static Route

To route traffic to the network 10.0.0.0/24 via gateway 10.0.0.1, use the following command:

ip route add 10.0.0.0/24 via 10.0.0.1

To make the route persistent, edit the /etc/sysconfig/network-scripts/route-ens192 file:

10.0.0.0/24 via 10.0.0.1 dev ens192

Restart the network:

systemctl restart network

# 4. Configure Network Address Translation (NAT)

4.1 Enable Packet Forwarding for NAT
Edit the /etc/sysctl.conf file:

net.ipv4.ip_forward = 1

Apply the changes:

sysctl -p

4.2 Configure NAT Using iptables

Set up NAT on the ens192 interface:

iptables -t nat -A POSTROUTING -o ens192 -j MASQUERADE

To make this rule persistent across reboots, install iptables-services if not already installed:

yum install -y iptables-services

service iptables save

# 5. Implement Quality of Service (QoS)

5.1 Define Traffic Class
Install the tc (Traffic Control) utility using iproute2:

yum install iproute2

Define a traffic class with a bandwidth rate limit of 1000 Mbit on ens192:

tc qdisc add dev ens192 root handle 1: htb default 12

tc class add dev ens192 parent 1: classid 1:1 htb rate 1000mbit

To make these rules persistent, you may need to add them to a script and execute it at startup.

# 6. Set Up VPN (Virtual Private Network)

6.1 Install OpenVPN

Install OpenVPN:

yum install -y epel-release

yum install -y openvpn

6.2 Generate SSL Certificates

Follow OpenVPN documentation to generate SSL certificates using easy-rsa or openssl.

6.3 Configure OpenVPN

Create an OpenVPN server configuration file in 
/etc/openvpn/server/server.conf and modify as per your network:

port 1194

proto udp

dev tun

server 10.8.0.0 255.255.255.0

push "route 10.0.0.0 255.255.255.0"

Start and enable OpenVPN:

systemctl start openvpn@server

systemctl enable openvpn@server

# 7. Implement Firewall Rules

7.1 Add Port 80 to Public Zone

To open port 80/tcp in the firewall, use the following commands:

firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --reload

# 8. Deploy Network Monitoring Tools

8.1 Install Zabbix

Install Zabbix and necessary components:

yum install -y zabbix-server-mysql zabbix-web-mysql zabbix-agent

8.2 Configure Zabbix

Follow Zabbix setup guide to configure the Zabbix server, agent, and web interface.

# 9. Enable IPv6

9.1 Assign IPv6 Address

To configure an IPv6 address on ens192, use the following command:

ip addr add 2001:0db8:85a3:0000:0000:8a2e:0370:7334/64 dev ens192

9.2 Enable IPv6 Forwarding

Edit /etc/sysctl.conf and enable IPv6 forwarding:

net.ipv6.conf.all.forwarding = 1

Apply the changes:

sysctl -p

# 10. Implement Dynamic Host Configuration Protocol (DHCP)

10.1 Install DHCP Server

Install the DHCP server:

yum install -y dhcp

10.2 Configure DHCP Server

Edit the DHCP configuration file /etc/dhcp/dhcpd.conf:

option domain-name "example.com";
option domain-name-servers 8.8.8.8, 8.8.4.4;
default-lease-time 600;
max-lease-time 7200;

subnet 10.0.0.0 netmask 255.255.255.0 {
  range 10.0.0.98 10.0.0.200;
  option routers 10.0.0.1;
  option subnet-mask 255.255.255.0;
  option broadcast-address 10.0.0.255;
}
Start and enable the DHCP service:

systemctl start dhcpd
systemctl enable dhcpd

# Quick Summary

Implement VLANs:

Use ip link add to create a VLAN subinterface on ens192 with VLAN ID 10.
Assign an IP address to the VLAN interface using ip addr add.
Set Up Bonding/Teaming:

Configure network bonding in /etc/sysconfig/network-scripts/. Set bond mode to active-backup for redundancy and configure slave interfaces.
Implement IP Routing:

Enable IP forwarding by editing /etc/sysctl.conf.
Add static routes with ip route to enable routing between subnets.
Configure NAT:

Use iptables to set up NAT for private IPs by masquerading traffic on ens192.

Implement QoS:

Use tc to create traffic classes and limit bandwidth to 1000mbit on ens192.
Set Up VPN:

Install OpenVPN, generate SSL certificates, and configure VPN to provide secure remote access.

Implement Firewall Rules:

Use firewall-cmd to add firewall rules, such as opening port 80 (HTTP).

Deploy Network Monitoring Tools:

Install and configure Zabbix to monitor network traffic and performance.

Enable IPv6:

Assign an IPv6 address to ens192 and enable IPv6 forwarding in 
/etc/sysctl.conf.

Implement DHCP:

Install a DHCP server and configure /etc/dhcp/dhcpd.conf to assign IP addresses to clients automatically.ria

VLANs, bonding/teaming, IP routing, NAT, QoS, VPN, firewall rules, DHCP, and IPv6 configurations have been implemented and tested.

Zabbix is deployed for network monitoring.

The CentOS server efficiently manages network traffic with enhanced security and reliable connectivity for client devices and services.

