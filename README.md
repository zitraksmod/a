---------------------------------------ISP-----------------------------------------------------

hostnamectl set-hostname isp
timedatectl set-timezone Europe/Moscow

nano /etc/sysctl.conf
net.ipv4.ip_forward = 1
sysctl -p


nano /etc/network/interfaces

auto ens18
iface ens18 inet dhcp
    post-up iptables -t nat -I POSTROUTING -o ens18 -j MASQUERADE
    post-down iptables -t nat -F

auto ens19
iface ens19 inet static
    address 172.16.40.1/28

auto ens20
iface ens20 inet static
    address 172.16.50.1/28

---------------------------------------HQ-RTR--------------------------------------------

hostnamectl set-hostname hq-rtr.au-team.irpo
timedatectl set-timezone Europe/Moscow

nano /etc/sysctl.conf
net.ipv4.ip_forward = 1
sysctl -p 

systemctl enable ssh
systemctl enable frr
systemctl enable dnsmasq


nano /etc/network/interfaces

auto ens18
iface ens18 inet static
        address 172.16.40.2/28
        gateway 172.16.40.1
        post-up iptables -t nat -I POSTROUTING -o ens18 -j MASQUERADE
        post-down iptables -t nat -F

auto ens19.10
iface ens19.10 inet static
        address 192.168.10.1/29
        vlan-raw-device ens19

auto ens19.20
iface ens19.20 inet static
        address 192.168.20.1/29
        vlan-raw-device ens19

auto ens19.99
iface ens19.99 inet static
        address 192.168.99.1/30
        vlan-raw-device ens19

auto gre1
iface gre1 inet static
    address 192.168.255.1
    netmask 255.255.255.252
    pre-up ip tunnel add gre1 mode gre remote 172.16.50.2 local 172.16.40.2 ttl 64 dev ens18
    up ip link set gre1 up
    post-down ip tunnel del gre1

useradd -m -s /bin/bash net_admin
passwd net_admin 
P@$$word
nano /etc/sudoers
net_admin ALL=(ALL) NOPASSWD: ALL

nano /etc/frr/daemons
ospfd=yes
systemctl restart frr.service
vtysh
conf t
interface gre1
ip ospf authentication message-digest
ip ospf message-digest-key 1 md5 P@ssw0rd
ip ospf network point-to-point
exit
router ospf
passive-interface default
no passive-interface gre1
network 192.168.10.0/29 area 0
network 192.168.20.0/29 area 0
network 192.168.99.0/30 area 0
network 192.168.255.0/30 area 0
area 0 authentication message-digest
do wr

echo "" > /etc/dnsmasq.conf
nano /etc/dnsmasq.conf
domain=au-team.irpo
interface=ens19.20
dhcp-range=192.168.20.2,192.168.20.6,24h
dhcp-option=1,255.255.255.248
dhcp-option=3,192.168.20.1
dhcp-option=6,192.168.10.2
dhcp-host=bc:24:11:84:40:b2,192.168.20.6


---------------------------------------BR-RTR-----------------------------------------------------
hostnamectl set-hostname br-rtr.au-team.irpo
timedatectl set-timezone Europe/Moscow

nano /etc/sysctl.conf
net.ipv4.ip_forward = 1
sysctl -p 

systemctl enable ssh
systemctl enable frr



nano /etc/network/interfaces

auto ens18
iface ens18 inet static
        address 172.16.50.2/28
        gateway 172.16.50.1
        post-up iptables -t nat -I POSTROUTING -o ens18 -j MASQUERADE
        post-down iptables -t nat -F

auto ens19 
iface ens19 inet static
        address 192.168.3.1/29

auto gre1
iface gre1 inet static
    address 192.168.255.2
    netmask 255.255.255.252
    pre-up ip tunnel add gre1 mode gre remote 172.16.40.2 local 172.16.50.2 ttl 64 dev ens18
    up ip link set gre1 up
    post-down ip tunnel del gre1

useradd -m -s /bin/bash net_admin
passwd net_admin 
P@$$word
nano /etc/sudoers
net_admin ALL=(ALL) NOPASSWD: ALL

nano /etc/frr/daemons
ospfd=yes
systemctl restart frr.service
vtysh
conf t
interface gre1
ip ospf authentication message-digest
ip ospf message-digest-key 1 md5 P@ssw0rd
ip ospf network point-to-point
exit
router ospf
passive-interface default
no passive-interface gre1
network 192.168.3.0/29 area 0
network 192.168.255.0/30 area 0
area 0 authentication message-digest
do wr
---------------------------------------HQ_SRV-----------------------------------------------------

hostnamectl set-hostname hq-srv.au-team.irpo
timedatectl set-timezone Europe/Moscow

systemctl enable ssh

nano /etc/network/interfaces
auto ens18
iface ens18 inet static
        address 192.168.10.2/29
        gateway 192.168.10.1


useradd -u 1015 -m -s /bin/bash sshuser
echo sshuser:P@ssw0rd | chpasswd
nano /etc/sudoers
sshuser ALL=(ALL) NOPASSWD: ALL


nano /etc/ssh/banner.txt
Authorized access only

nano etc/ssh/sshd_config.d/demo.conf

Port 3015
AllowUsers sshuser
MaxAuthTries 2
Banner /etc/ssh/banner.txt


---------------------------------------BR-SRV-----------------------------------------------------

hostnamectl set-hostname br-srv.au-team.irpo
timedatectl set-timezone Europe/Moscow

systemctl enable ssh

nano /etc/network/interfaces
auto ens18
iface ens18 inet static
        address 192.168.3.2/29
        gateway 192.168.2.1


useradd -u 1015 -m -s /bin/bash sshuser
echo sshuser:P@ssw0rd | chpasswd
nano /etc/sudoers
sshuser ALL=(ALL) NOPASSWD: ALL

nano /etc/ssh/banner.txt
Authorized access only

nano etc/ssh/sshd_config.d/demo.conf

Port 3015
AllowUsers sshuser
MaxAuthTries 2
Banner /etc/ssh/banner.txt

--------------------------------------HQ-CLI-----------------------------------------------------


hostnamectl set-hostname hq-cli.au-team.irpo
timedatectl set-timezone Europe/Moscow


