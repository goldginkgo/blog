---
title: Setup Concourse in Ubuntu from scratch with Proxy
date: 2018-07-03 21:19:32
categories: DevOps
tags:
  - Concourse
---
Concourse is a greate tool for CI/CD.
Here is a guide to setup Concourse on Ubuntu 18.04 from scratch in Corporate environment.
<!-- more -->
### Install Ubuntu 18.04.

### Check DNS and hostname
```
$ sudo vim /etc/resolv.conf
nameserver <dns-server>

$ sudo vi /etc/hosts
127.0.0.1       localhost.localdomain   localhost frankvm
```

### Install cntlm
```
Set temporary proxy in following file for installing cntlm
$ sudo touch /etc/apt/apt.conf.d/proxy.conf
$ sudo vi /etc/apt/apt.conf.d/proxy.conf
Acquire::http::Proxy "http://dfr1szh:<password>@rb-proxy-apac.bosch.com:8080/";
Acquire::https::Proxy "https://dfr1szh:<password>@rb-proxy-apac.bosch.com:8080/";

Enable universe repository so that we can install cntlm.
$ sudo add-apt-repository universe

check repository
$ apt policy

$ cat /etc/apt/sources.list

install cntlm
$ sudo apt-get install cntlm

cntlm configuration
$ cntlm -u <user> -d <domain> -H
PassNTLMv2      67DB3F227E0A9136A7C5D8D0532A1AAB

$ sudo vim /etc/cntlm.conf
Username        <user>
Domain          <domain>
#Password       password
PassNTLMv2      67DB3F227E0A9136A7C5D8D0532A0AAB
Proxy           <proxy-ip>:8080
NoProxy         localhost, 127.0.0.*, 10.*, 192.168.*
Listen          <local-ip>:3128

$ sudo systemctl restart cntlm
```

### Set proxy
replace <local-ip> with actual ip
 ```
$ sudo vi /etc/environment
http_proxy=http://<local-ip>:3128/
ftp_proxy=http://<local-ip>:3128/
socks_proxy=socks://<local-ip>:3128/
https_proxy=http://<local-ip>:3128/
no_proxy=127.0.0.1,localhost,10.*,192.168.*,*.bosch.com

$ source /etc/environment

$ sudo vi /etc/apt/apt.conf.d/proxy.conf
Acquire::http::proxy "http://<local-ip>:3128/";
Acquire::https::proxy "http://<local-ip>:3128/";

reboot if necessary
```

### Install Docker

https://docs.docker.com/install/linux/docker-ce/ubuntu/

https://docs.docker.com/install/linux/linux-postinstall/#next-steps

https://docs.docker.com/config/daemon/systemd/#httphttps-proxy

```
sudo vi /etc/systemd/system/docker.service.d/http-proxy.conf
[Service]
Environment="HTTP_PROXY=http://<local-ip>:3128/"
Environment="HTTPS_PROXY=http://<local-ip>:3128/"
Environment="NO_PROXY=localhost,127.0.0.*,10.*,192.168.*,*.bosch.com"
```

### Install certificates
When using tools like wget or docker to download HTTPS secured content, need to install trusted self-signed proxy certs.
```
Extract certs to /usr/local/share/ca-certificates
$ sudo update-ca-certificates
```

### Install [Docker Compose](https://docs.docker.com/compose/install/#install-compose)


### Install [Ruby](https://www.digitalocean.com/community/tutorials/how-to-install-ruby-on-rails-with-rbenv-on-ubuntu-18-04)
(not relevant)

### Install Concourse
```
$ git clone https://github.com/concourse/concourse-docker.git && cd concourse-docker
$ sudo ./generate-keys.sh
$ cat docker-compose-quickstart.yml
version: '3'

services:
  concourse-db:
    image: postgres
    environment:
    - POSTGRES_DB=concourse
    - POSTGRES_PASSWORD=concourse_pass
    - POSTGRES_USER=concourse_user
    - PGDATA=/database

  concourse-web:
    image: concourse/concourse:4.2.1
    command: web
    links: [concourse-db]
    depends_on: [concourse-db]
    ports: ["8080:8080"]
    volumes: ["./keys/web:/concourse-keys"]
    environment:
    - CONCOURSE_POSTGRES_HOST=concourse-db
    - CONCOURSE_POSTGRES_USER=concourse_user
    - CONCOURSE_POSTGRES_PASSWORD=concourse_pass
    - CONCOURSE_POSTGRES_DATABASE=concourse
    - CONCOURSE_EXTERNAL_URL=http://<local-ip>:8080
    - CONCOURSE_ADD_LOCAL_USER=test:$$2a$$10$$0W9/ilCpYXY/yCPpaOD.6eCrGda/fnH3D4lhsw1Mze0WTID5BuiTW
    - CONCOURSE_MAIN_TEAM_ALLOW_ALL_USERS=true
    - http_proxy=http://<local-ip>:3128
    - https_proxy=http://<local-ip>:3128
    - no_proxy=127.0.0.1,localhost,10.*,192.168.*


  concourse-worker:
    image: concourse/concourse:4.2.1
    command: worker
    privileged: true
    links: [concourse-web]
    depends_on: [concourse-web]
    volumes: ["./keys/worker:/concourse-keys"]
    environment:
    - CONCOURSE_TSA_HOST=concourse-web:2222
    - CONCOURSE_GARDEN_NETWORK
    - http_proxy=http://<local-ip>:3128
    - https_proxy=http://<local-ip>:3128
    - no_proxy=127.0.0.1,localhost,10.*,192.168.*

$ docker-compose up -d
$ sudo curl -L "https://github.com/concourse/concourse/releases/download/v4.2.1/fly_linux_amd64" -o /usr/local/bin/fly
$ sudo chmod +x /usr/local/bin/fly
```
