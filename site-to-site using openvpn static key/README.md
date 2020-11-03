OpenVPN site to site with centos7 and symmetric encryption

OFFICE:
Network: 192.168.10.0/24

HOME:
Network: 192.168.20.0/24


DO THIS ON ALL MACHINES:

yum install https://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-5.noarch.rpm
yum install openvpn

DO THIS ON OFFICE MACHINE:

vi /etc/openvpn/office-home.conf
------
remote home.compress.to
port 4001
float
proto udp
dev tun1
ifconfig 172.10.0.1 172.10.0.2
persist-tun
persist-local-ip
persist-remote-ip
comp-lzo
ping 15
secret /etc/openvpn/office-home.key
route 192.168.20.0 255.255.255.0
user openvpn
group openvpn
syslog office-home
verb 1
------

vi /etc/sysconfig/iptables
------
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
# openvpn
-A INPUT -p udp --dport 8001 -j ACCEPT
# do not allow anything else
-A INPUT -j REJECT --reject-with icmp-host-prohibited
# openvpn
-A FORWARD -s 192.168.10.0/24 -d 192.168.20.0/24 -j ACCEPT
-A FORWARD -s 192.168.20.0/24 -d 192.168.10.0/24 -j ACCEPT
# do not allow anything else
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
------

openvpn --genkey --secret /etc/openvpn/office-home.key
chmod 600 /etc/openvpn/office-home.conf
chmod 400 /etc/openvpn/office-home.key
scp /etc/openvpn/office-home.key root@vpn-home:/etc/openvpn/office-home.key

echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf
sysctl -p

systemctl enable iptables
systemctl restart iptables

systemctl enable openvpn@office-home
systemctl restart openvpn@office-home

DO THIS ON HOME MACHINE:

vi /etc/openvpn/home-office.conf
------
remote office.compress.to
port 4001
float
proto udp
dev tun1
ifconfig 172.10.0.2 172.10.0.1
persist-tun
persist-local-ip
persist-remote-ip
comp-lzo
ping 15
secret /etc/openvpn/office-home.key
route 192.168.10.0 255.255.255.0
user openvpn
group openvpn
syslog office-home
verb 1
------

vi /etc/sysconfig/iptables
------
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
# openvpn
-A INPUT -p udp --dport 8001 -j ACCEPT
# do not allow anything else
-A INPUT -j REJECT --reject-with icmp-host-prohibited
# openvpn
-A FORWARD -s 192.168.10.0/24 -d 192.168.20.0/24 -j ACCEPT
-A FORWARD -s 192.168.20.0/24 -d 192.168.10.0/24 -j ACCEPT
# do not allow anything else
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
------

chmod 600 /etc/openvpn/home-office.conf
chmod 400 /etc/openvpn/home-office.key

echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf
sysctl -p

systemctl enable iptables
systemctl restart iptables

systemctl enable openvpn@home-office
systemctl restart openvpn@home-office
