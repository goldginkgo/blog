---
title: Setup Shadowsocks to Visit Google in China
date: 2018-02-08 19:18:08
tags:
  - VPN
---
Here is a guilde on how to setup Shadowssocks to visit blocked websites like Google in China.
<!-- more -->
### Register for a VPS provider
I will take [Vultr](https://www.vultr.com) for example.

### Upload your SSH key to Vultr
An SSH Key allows you to log into your server without needing a password. SSH Keys can be automatically added to servers during the installation process.

### Create a Ubuntu 18.04 VM
Create a Ubuntu 18.04 VM in your VPS with your SSH key specified, and set location to places where have access to Google. (e.g. Los Angeles)

### [Disable ssh timeout](http://queirozf.com/entries/disabling-ssh-timeout-when-connecting-to-from-ubuntu).

### Install shadowsocks service on VPS host
Execute the following script, or add this script as startup script when creating the vm. Logs for shadowsocks are in /var/log/shadowsocks.log.
``` bash
#!/bin/sh

# Deploy Ubuntu 18.04 VM

#echo -e "ClientAliveInterval 600\nTCPKeepAlive yes\nClientAliveCountMax 10" >> /etc/ssh/sshd_config
apt-get update
apt-get install python-pip -y
apt-get install -y python-setuptools
pip install git+https://github.com/shadowsocks/shadowsocks.git@master
#pip install shadowsocks

sudo ufw disable

IP=$(hostname -I)
cat > /etc/shadowsocks.json <<EOF
{
    "server": "$IP",
    "server_port": "8899",
    "local_address": "127.0.0.1",
    "local_port": 1080,
    "password": "asdf_1234",
    "timeout": 300,
    "method": "aes-256-cfb",
    "fast_open": true
}
EOF

# TCP BBR
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
sysctl net.ipv4.tcp_available_congestion_control

ssserver -c /etc/shadowsocks.json -d start

```

### Setup client on your laptop
Download [MacOS Client](https://github.com/shadowsocks/ShadowsocksX-NG/releases), install, and enter server IP, port, password information in client. Change shadowsocks "HTTP proxy listen ip" to local ip. e.g 192.168.1.107. use 'ipconfig getifaddr en0' command to get IP.

### Use shadowsocks proxy in command line
``` bash
export http_proxy=http://192.168.1.107:1087;export https_proxy=http://192.168.1.107:1087;export no_proxy=localhost,127.0.0.0/8,192.0.0.0/8;
```

Reference:

[A Chinese Guide](https://www.flyzy2005.cn/fan-qiang/shadowsocks/build-shadowsocks-on-vps/)

[shadowsocks Official Website](https://shadowsocks.org/en/index.html)

[shadowsocks Github](https://github.com/shadowsocks)

[shadowsocks wiki](https://github.com/shadowsocks/shadowsocks/wiki)
