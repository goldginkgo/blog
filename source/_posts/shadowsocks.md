---
title: Set up Shadowsocks to Visit Google
date: 2018-02-08 19:18:08
tags:
---
1. Register for a cheap VPS provider. I will take [Vultr](https://www.vultr.com) for example.
2. Upload your SSH key to Vultr. An SSH Key allows you to log into your server without needing a password. SSH Keys can be automatically added to servers during the installation process.
2. Create a Ubuntu host in your VPS with your SSH key specified, and set location to places where have access to Google. (e.g. Los Angeles)
3. Login to VPS host and install shadowsocks service. [Disable ssh timeout](http://queirozf.com/entries/disabling-ssh-timeout-when-connecting-to-from-ubuntu).  
``` bash
# ssh -i ~/.ssh/id_rsa root@<vps-ip>
# apt-get install python-pip -y
# wget https://bootstrap.pypa.io/ez_setup.py -O - | sudo python
# pip install setuptools --upgrade
# pip install shadowsocks
# iptables -F
```
4. Create a new configuration file for shadowsocks.
```
# vi /etc/shadowsocks.json
{
    "server": "<vps-ip>",
    "server_port": <your-port-number(e.g.8899)>,
    "local_address": "127.0.0.1",
    "local_port": 1080,
    "password": "<shadows-password-for-client>",
    "timeout": 300,
    "method": "aes-256-cfb",
    "fast_open": true
}
```
5. Start/Stop shadowsocks in the background.  
Logs for shadowsocks are in /var/log/shadowsocks.log.
``` bash
ssserver -c /etc/shadowsocks.json -d start
ssserver -c /etc/shadowsocks.json -d stop
```

6. Downlocad [MacOS Client](https://github.com/shadowsocks/ShadowsocksX-NG/releases), install, and enter server IP, port, password information in client.
7. If you want to use shadowsocks proxy in command line:
``` bash
export http_proxy=http://127.0.0.1:1087
export https_proxy=http://127.0.0.1:1087
```

Reference:  
[A Chinese Guide](https://www.flyzy2005.cn/fan-qiang/shadowsocks/build-shadowsocks-on-vps/)  
[shadowsocks Official Website](https://shadowsocks.org/en/index.html)  
[shadowsocks Github](https://github.com/shadowsocks)  
[shadowsocks wiki](https://github.com/shadowsocks/shadowsocks/wiki)
