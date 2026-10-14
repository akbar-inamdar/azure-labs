# Day 1: Azure basics (VM, VNet, NSG)

## What I built
A resource group, a virtual network with one subnet, and an Ubuntu VM running nginx, first in the portal and then with the Azure CLI. I deleted everything afterwards.

## Commands I ran in Cloud Shell
```
az group create -n rg-day1cli -l centralindia
az network vnet create -g rg-day1cli -n vnet-cli --address-prefix 10.0.0.0/16 --subnet-name web --subnet-prefix 10.0.1.0/24
az vm create -g rg-day1cli -n vm-cli --image Ubuntu2204 --size Standard_B2ts_v2 --security-type Standard --vnet-name vnet-cli --subnet web --admin-username azureuser --generate-ssh-keys
az vm open-port -g rg-day1cli -n vm-cli --port 80
az vm run-command invoke -g rg-day1cli -n vm-cli --command-id RunShellScript --scripts "sudo apt-get update && sudo apt-get install -y nginx"
az group delete -n rg-day1cli --yes --no-wait
```

## What I learned (in my own words)
- Resource group: Resource group is a folder or container which is used to store the related resources at a single group, like the vm, disk, network, etc in same group and I can delete the whole project by deleting the group
- VNet and subnet: Vnet is a whole private virtual network while subnet is a part/slice of it, /16 means it fixes 16 bits and remaining 16 bits are free which gives 65536 addresses, Azure reserves 5 of them, so 65531 are usable and /24 means it fixes 24 bits and remaining 8 bits are free which gives 256 addresses, Azure reserves 5 of them, so 251 are usable
- NSG: A Network Security Group is a list of allow and deny rules that controls traffic to a VM. Rules are checked by priority, lowest number first, and the first match wins. Azure has a default rule (priority 65500) that denies all inbound internet traffic, so nginx only became reachable after I added an allow rule for port 80.
## Problems I hit and how I fixed them
- Quota error on the VM size: I first picked Standard_B2ats_v2 because it was the free-tier size, but the portal said I had 0 vCPUs available for that VM family in Central India. I hadn't realised new subscriptions can start with zero quota for some sizes. I switched to Standard_B2ts_v2, which had quota, and the VM deployed fine. Next time I'll check the quota page (Quotas in the portal) before choosing a size.
- Trusted Launch image error: While creating the VM in the portal, I got a red error saying the Ubuntu image wasn't compatible with the Trusted launch security type. I fixed it by changing the security type to Standard. When I redid the lab in the CLI, I added --security-type Standard to the command from the start, so it didn't happen again.
