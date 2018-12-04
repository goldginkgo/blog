---
title: How to find apps with high CPU usage for a Diego cell VM.
date: 2018-06-30 18:56:10
categories: Cloud
tags:
  - Cloud Foundry
---

According to [Understanding Container Security](https://docs.cloudfoundry.org/concepts/container-security.html), each container in Cloud Foundry use a fair share of CPU relative to the other containers. However, an app container can use much higher CPU resources when other containers in the same Diego Cell VM are idle if we don't set limitation for CPU usage of app containsers.
<!-- more -->
It would be rather difficult to figure out the apps that is causing the trouble if we don't have application level monitoring.

One way to achieve our goal is to use Cloud Foundry [top](https://github.com/ECSTeam/cloudfoundry-top-plugin) plugin. But here I also want to show the procedures on how to do it manually.

Before we actually start, I want to give some steps on how to find the Diego Cell VM information based on an application instace.

## Mapping of Diego Cell and Cloud Foundry apps
### Get mapping information of Diego Cell and Cloud Foundry apps
cf login to your org and space, use commands below to get IP address of the cell. CPU and Memory usage is also shown in this command.
```
$ cf app web-app-bb --guid
c07cf94e-3c43-41b6-9e1b-e77231fa2058
$ cf curl /v2/apps/c07cf94e-3c43-41b6-9e1b-e77231fa2058/stats
{
   "0": {
      "state": "RUNNING",
      "isolation_segment": null,
      "stats": {
         "name": "web-app-bb",
         "uris": [
            "web-app-bb.apps.<url>"
         ],
         "host": "10.48.16.166",
         "port": 61001,
         "uptime": 114,
         "mem_quota": 67108864,
         "disk_quota": 1073741824,
         "fds_quota": 16384,
         "usage": {
            "time": "2018-12-02T08:25:52+00:00",
            "cpu": 0.0036034101653391625,
            "mem": 16740352,
            "disk": 185126912
         }
      }
   }
}
```
### SSH to Diego cell VM
Login to Bosh, and execute the following command to ssh to the Diego cell VM.
```
$ bosh -e pcf log-in
$ bosh -e pcf -d cf-c16bf7a74c8a95e1e6eb vms | grep 10.48.16.166
diego_cell/9dd712e7-31d2-4d44-845f-91bd7495e9bd                         running PCF-QA-AZ3  10.48.16.166    vm-dd56c1cc-1edf-44a7-81ff-6ce22a32d9dc ert.custom2     -
$ bosh -e pcf -d cf-c16bf7a74c8a95e1e6eb ssh diego_cell/9dd712e7-31d2-4d44-845f-91bd7495e9bd
```

Now let's go on with the steps to find applications based on the Deigo cell VM.
## Find applications on a Deigo cell VM
### Check stats on Diego cell
Login to Diego cell VM and check the stats on the VM. You will get resulst like following, first half of process_guid is the actuall App GUID.
```
diego_cell/9dd712e7-31d2-4d44-845f-91bd7495e9bd:~$ cd /var/vcap/jobs/rep/config/certs; curl -k -s https://localhost:1801/state --cert server.crt --key server.key | python -m json.tool
{
    "AvailableResources": {
        "Containers": 236,
        "DiskMB": 173866,
        "MemoryMB": 52485
    },
    "Evacuating": false,
    "LRPs": [
        {
            "DiskMB": 1024,
            "MaxPids": 0,
            "MemoryMB": 1024,
            "PlacementTags": null,
            "RootFs": "preloaded:cflinuxfs2",
            "VolumeDrivers": [],
            "domain": "cf-apps",
            "index": 0,
            "instance_guid": "30525d4f-81ab-4cf3-7c61-bb6f",
            "process_guid": "75a52aba-3eab-429a-ad6f-160619241e2d-55c2deda-cf67-4d2b-9f1c-8daf2bbeeea0"
        },
...
    "TotalResources": {
        "Containers": 249,
        "DiskMB": 184938,
        "MemoryMB": 64429
    },
...
```

### Get all app GUIDs on Diego cell
```
diego_cell/9dd712e7-31d2-4d44-845f-91bd7495e9bd:/var/vcap/jobs/rep/config/certs$ cd /var/vcap/jobs/rep/config/certs; curl -k -s https://localhost:1801/state --cert server.crt --key server.key | python -m json.tool | grep "process_guid" |cut -d ':' -f2|cut -c 3-38
c07cf94e-3c43-41b6-9e1b-e77231fa2058
656cd296-b2f4-4c4d-a82d-8079763d3e8a
5f159b3d-6e61-4d00-9fca-deee3f035c57
40b135af-c7c7-4ebd-af2e-984b2b5ce262
382ce123-c13c-472f-ab74-5265dc984714
2044e186-63bd-494e-8559-e28fd3e35e28
0409effc-f11a-4dc7-a6e4-15282e234a53
fcf52090-7c8a-45db-90f6-29f25fa0be53
048b4d26-4652-4c85-8015-4bb2be42aeff
8e3ba265-a0df-4b07-a34a-aaea3158e090
75a52aba-3eab-429a-ad6f-160619241e2d
d048268a-e60b-481a-a5cc-ff39fdbf1eb2
e7428432-b459-4111-b813-4fb6dfc7884f
```

### Get apps with CPU info
Login to Cloud Foundry with admin account and using following shell script to get apps names. app_guids is a file with contents from previous step.
```
filename="app_guids"
cat $filename | while read LINE
do
   echo $LINE
   cf curl /v2/apps/$LINE | grep "\"name\":"
   cf curl /v2/apps/$LINE | grep "\"cpu\":"
done
```
Here is a sample result:
```
$ sh script
c07cf94e-3c43-41b6-9e1b-e77231fa2058
      "name": "web-app-bb",
            "cpu": 0.0026490531155109625,
656cd296-b2f4-4c4d-a82d-8079763d3e8a
      "name": "connection-test",
            "cpu": 0.0008588435074484718,
            "cpu": 0.0009525608833325616,
0409effc-f11a-4dc7-a6e4-15282e234a53
      "name": "apps-manager-js-blue",
            "cpu": 7.13585021377469e-06,
            "cpu": 8.020481371308023e-06,
            "cpu": 7.091829884025554e-06,
            "cpu": 0.0,
            "cpu": 0.0,
            "cpu": 0.0,
...
```