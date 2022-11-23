# create-router-with-iptables-on-linux

___***INTRO***___\n
***YOU NEED TO LEARN IPTABLES***\n
***Right NOW!!***\n
But why?...
Well iptables can create router with one command!!

In this thread I'm going to teach you ,how to create ***ROUTER*** using iptables and how to do to portforward on it.

___***What is IPTABLES?***___
***IPTABLES*** is a packet filters installed in general Linux.
***IPTABLES*** is a very sophisticated and high-performance packet filter comparable to commercial products.
In addition to protecting the server itself, it also supports packet forwarding.
So it can be used as a network firewall by running on a machine equipped with two network adapters to control packet forwarding.
🙌 Thank you for using it for free!

___***・First Step***___
This thread uses <:ubuntu:942695210847195176> ubuntu as an example. 
However, ***IPTABLES*** is a <:linux:942698642429603880>Linux feature, so it works exactly the same way on other Distributions.

Need to allow *forward* of ipv4.(If you want to use ipv6 too, you need check out it too.)
```bash
sudo bash -c "cat /etc/sysctl.conf | sed -e 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/g' > /etc/new_sysctl.conf && rm /etc/sysctl.conf && mv /etc/new_sysctl.conf /etc/sysctl.conf && sysctl -p"
sudo bash -c "modprobe ip_nat_ftp; modprobe ip_conntrack; modprobe ip_conntrack_ftp; modprobe ip_tables; modprobe iptable_nat; modprobe ipt_MASQUERADE"
```
Install a package to permanently store iptables data.
```bash
sudo apt install iptables-persistent -y
```

___***・Second Step***___
Configure router settings.
This is very easy and takes only one line. 
```bash
# Example: Using 192.168.20.0/24
sudo iptables -t nat -A POSTROUTING -s 192.168.20.0/24 -o ppp0 -j MASQUERADE
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```With the above line, the router is now complete.
However, this will reset upon reboot.
So, we will now make it permanent.
```bash
sudo bash -c "iptables-save > /etc/iptables/rules.v4"
```

___***・Finally***___
This step describes port-forwarding.
```bash
# Example: forward 80 to 192.168.20.101:8080
sudo iptables -t nat -A PREROUTING ! -s 192.168.20.0/24 -p tcp -m tcp --dport 80 -j DNAT --to-destination 192.168.20.101:8080
sudo bash -c "iptables-save > /etc/iptables/rules.v4"
```**Q:**` ! -s 192.168.20.0/24` <- what is this?
**A:** This means to target the port for communication from other than 192.168.20.0/24.If this is not stated, port 80 will internally loop back to port 8080.


___***・Issue***___
In rare cases, when you try to browse a particular website, you may not be able to do so. 
That is because the **MTU** is not an accurate value.
You can correct the **MTU** with the following command.
```bash
sudo iptables -A FORWARD -p tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu
sudo bash -c "iptables-save > /etc/iptables/rules.v4"
```

Thank you for read my docs!
