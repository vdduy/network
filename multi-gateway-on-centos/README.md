# Multi gateway on Centos - Reply on same interface as incoming

I'm assuming you are running Linux and, further, that you are utilising a RedHat/CentOS-based distribution. Other Unix's and distributions will require similar steps - but the details will be different.

Start by testing (note that this is very similar to @Peter's answer. I am assuming the following:
```
    eno0 is isp0 and has the overall default gateway
    eno1 is isp1 and has the IP/range 192.168.1.2/24 with gateway 192.168.1.1
```
The commands are as follows:
```
echo 200 isp1 >> /etc/iproute2/rt_tables
ip rule add from 192.168.1.2 table isp1
ip route add default via 192.168.1.1 dev eno1 table isp1
```
   
 
The firewall is not involved in any way. Reply packets would always have been sent from the correct IP - but previously were being sent out via the wrong interface. Now these packets from the correct IP will be sent via the correct interface.

Assuming the above worked, you can now make the rule and route changes permanent. This depends on what version of Unix you are using. As before, I'm assuming a RH/CentOS-based Linux distribution.
```
$ echo "from eno1 table isp1" > /etc/sysconfig/network-scripts/rule-eno1
$ echo "default via 192.168.1.1 dev eno1 table isp1" > /etc/sysconfig/network-scripts/route-eno1
```
Test that the network change is permanent:
```
$ ifdown eno1 ; ifup eno1
```
If that didn't work, on the later versions of RH/CentOS you also need to do go with one of two options:

Don't use the default NetworkManager.service; Use network.service instead. I haven't explored the exact steps needed for this. I would imagine it involves the standard chkconfig or systemctl commands to enable/disable services.

Install the NetworkManager-dispatcher-routing-rules package

Personally I prefer installing the rules package as it is the simpler more supported approach:
```
$ yum install NetworkManager-dispatcher-routing-rules
```
Another strong recommendation is to enable arp filtering as this prevents other related issues with dual network configurations. With RH/CentOS, add the following content to the /etc/sysctl.conf file:
```
net.ipv4.conf.default.arp_filter=1
net.ipv4.conf.all.arp_filter=1
```

source: https://unix.stackexchange.com/questions/4420/reply-on-same-interface-as-incoming
