---
title: How to Use Cloud Foundry Services From outside the Cloud Envivonment
date: 2018-08-17 20:40:10
categories: Cloud
tags:
  - Cloud Foundry
---
Based on NGINX buildpack and TCP routing feature in Cloud Foundry, we can implement an NGINX app on cloud foundry which acts as a reverse proxy for Cloud Foundry services.

By accessing the app, we will be able to use Cloud Foundry services (RabbitMQ, Redis, ...) from outside Cloud Foundry.
<!-- more -->
Let's take RabbitMQ/AMQP protocol for example, here are the steps:

### Add RadditMQ service in Cloud Foundry
Login to your org and space in Cloud Foundry, add a RabbitMQ service instance.

### Create TCP route
Create a TCP route with the following command, and get TCP port information using "cf routes" command.
```
cf create-route <space-name> tcp1.<cloud-foundry-domain> --random-port
```

### Configure nginx.conf
Create a directory on your laptop. Create nginx.conf file in the directory with following contents.
```
{{module "ngx_stream_module"}}

worker_processes 1;
daemon off;
 
error_log stderr;
events { worker_connections 1024; }
 
stream {
    server {
        listen {{port}};
        proxy_pass {{env "RHOST"}}:5672;
   }
}
```

### Create app's manifest
Create manifest.yml file in the directory with the following content. Replace the values with actual ones. Just replace <rabbitmq-host-url> with 127.0.0.1 if you don't have the IP/FQDN for RabbitMQ.
```
---
applications:
- name: <app-name>
  buildpack: https://github.com/cloudfoundry/nginx-buildpack.git
  routes:
  - route: tcp1.<region>.bosch-iot-cloud.com:<tcp-port>
  env:
    RHOST: <rabbitmq-host-url>
  services:
  - <rabbitmq-service-name>
```
### Push app
```
cf push
```

### Change RabbitMQ setting in manifest.yml
Get RabbitMQ hostname from apps manager UI and change "RHOST" with actual value in manifest.yml and execute “cf push” again. Skip this if you already entered the right value in step 4.

### Whitelist the route
Whitelist the TCP route for you app to use if necessary.

Now you can develop apps which will be deployed outside Cloud Foundry to use RabbitMQ service in Cloud Foundry. Just get RabbitMQ connection information from Cloud Foundry, and replace RabbitMQ hostname and port with your NGINX app's tcp route. However, normally this is not allowed as we may have security concerns on using service outside Cloud Foundry.
