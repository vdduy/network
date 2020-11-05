# OpenVPN site to site with static key
Topology
```
                        Office
                       /  |  \
                  Home01 ...  Home n
```        
Là mô hình VPN site to site dạng hub-spoke. Tất cả spoke sẽ tập trung vào 1 hub.

OFFICE:
Local Network: 192.168.10.0/24
Tunnel1 (virtual IP): 172.16.0.1

HOME:
Local Network: 192.168.20.0/24
Tunnel1 (Virtual IP): 172.16.0.2

Nếu trên Office kết nối tới nhiều Home khác thì phải tạo Tunnel khác và range ip khác. 
Ví dụ IP Tunnel2 172.16.1.1


# Cài đặt openvpn trên offce và home

```
yum install epel-release -y
yum update -y
yum install openvpn -y
```

# Cài đặt trên Office
```
vi /etc/openvpn/office-home.conf
------
remote home.compress.to //Địa chỉ ip public của home
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
ping-restart 45
secret /etc/openvpn/office-home.key
cipher AES-256-CBC
route 192.168.20.0 255.255.255.0
user openvpn
group openvpn
syslog office-home
verb 1
------
```

Tạo static key
```
openvpn --genkey --secret /etc/openvpn/office-home.key
```
```
chmod 600 /etc/openvpn/office-home.conf
chmod 400 /etc/openvpn/office-home.key
```
copy static key sang home
```
scp /etc/openvpn/office-home.key root@vpn-home:/etc/openvpn/office-home.key
```
Cho phép routing trên tất cả interface. Mặc định CentOS không cho phép routing giữa các interface.
```
echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf
sysctl -p
```
Tương ứng với file config office-home.conf thì sẽ cần phải start service openvpn@office-home.
Nếu có nhiều home kết nối vào thì phải tạo riêng từng config cho mỗi home. Rồi start service openvpn@ tương ứng.
```
systemctl enable --now openvpn@office-home
```

# Cài đặt trên home

```
vi /etc/openvpn/home-office.conf
------
remote office.compress.to //Địa chỉ ip public của office
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
ping-restart 45
secret /etc/openvpn/office-home.key
cipher AES-256-CBC
route 192.168.10.0 255.255.255.0
user openvpn
group openvpn
syslog office-home
verb 1
------
```

```
chmod 600 /etc/openvpn/home-office.conf
chmod 400 /etc/openvpn/home-office.key
```
```
echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf
sysctl -p
```

```
systemctl enable openvpn@home-office
systemctl restart openvpn@home-office
```
