# tanzulab

See [vSphere with Tanzu Quick Start Guide V1a](https://core.vmware.com/resource/vsphere-tanzu-quick-start-guide-v1a#_Toc53677546)
See [Deploy HA-Proxy for vSphere with Tanzu](https://cormachogan.com/2020/09/25/deploy-ha-proxy-for-vsphere-with-tanzu/)

## Getting started

Please adjust the sections "Management network" and "Workload network" according your setup.

## Setup

### Management network

| IP | Hostname | Use |
| :- | :-: | :- |
| 192.168.1.1 | | Gateway |
| 192.168.1.180 | vcsa.lab.leyux.org | VCSA |
| 192.168.1.181 | esx01.wn.leyux.org | ESX01 |

### Workload network

| IP | Hostname | Use | Remark |
| :- | :-: | :- | :- |
| 192.168.11.1 | | Gateway | |
| 192.168.11.2 | | Load Balancer | |
| 192.168.11.8/29| | Cluster Node Range | 192.168.11.9-15 |
| 192.168.11.128/27 | | Virtual IP Range | 192.169.11.129-159 |


## Config

### Additional DvSwitch for workloads
```
$workloadhosts = get-cluster "lab" | get-vmhost
#Create the VDS Switch. MTU of 9000 is not necessary.
New-VDSwitch -Name "Dswitch" -MTU 1500 -NumUplinkPorts 1 -location Basement
Get-VDSwitch "Dswitch" | Add-VDSwitchVMHost -VMHost $workloadhosts
Get-VDSwitch "Dswitch" | Add-VDSwitchPhysicalNetworkAdapter -VMHostNetworkAdapter ($workloadhosts | Get-VMHostNetworkAdapter -Name vmnic3) -Confirm:$false
New-VDPortgroup -Name "Workload Network" -VDSwitch "Dswitch"
```

### Tag storage
```
# Set up tags for vSphere with Tanzu
# Adjust $datastore according to your setup
$datastore = "esx01_datastore01"
$StoragePolicyName = "kubernetes-demo-storage"
$StoragePolicyTagCategory = "kubernetes-demo-tag-category"
$StoragePolicyTagName = "kubernetes-gold-storage-tag"
New-TagCategory -Name $StoragePolicyTagCategory -Cardinality single -EntityType Datastore
New-Tag -Name $StoragePolicyTagName -Category $StoragePolicyTagCategory
Get-Datastore -Name $datastore | New-TagAssignment -Tag $StoragePolicyTagName
New-SpbmStoragePolicy -Name $StoragePolicyName -AnyOfRuleSets (New-SpbmRuleSet -Name "wcp-ruleset" -AllOfRules (New-SpbmRule -AnyOfTags (Get-Tag $StoragePolicyTagName)))
```

### Content library
```
#Set up the content library needed by vSphere with Tanzu
New-ContentLibrary -Datastore $datastore -name "tkg-cl" -AutomaticSync -SubscriptionUrl "http://wp-content.vmware.com/v2/latest/lib.json" -Confirm:$false
```

### HAProxy

Download [haproxy](https://cdn.haproxy.com/download/haproxy/vsphere/ova/vmware-haproxy-v0.1.8.ova)

Create Content Library
```
New-ContentLibrary -Datastore $datastore -name "HA-Proxy" -Confirm:$false
```

Upload haproxy through webui.
