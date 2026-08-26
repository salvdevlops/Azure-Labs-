Implement Intersite Connectivity

This lab is about getting two separate virtual networks to actually talk to each other — using virtual network peering, testing connectivity with Network Watcher, and adding a custom route to control traffic flow.

Overview

In this lab I created two VMs, each sitting in its own virtual network, confirmed they couldn't reach each other by default, set up VNet peering between the two networks, then retested and confirmed connectivity worked once peering was in place. Last step was creating a custom route table to direct traffic from a perimeter subnet through a (hypothetical) network virtual appliance before it reaches the core network.

Lab Scenario

The org keeps its core IT services (DNS, security, etc.) segmented from other parts of the business — in this case, manufacturing. Most of the time that separation is fine, but every so often something in core services needs to talk to something in manufacturing. This is basically the same pattern you'd see separating production from development, or splitting one subsidiary's network off from another's — separate by default, but with a controlled path to connect them when needed.

Environment:
Azure subscription

Diagram

[add diagram later]


Task 1: Create a Core Services VM and Virtual Network

First VM went into a brand new CoreServicesVnet network, created at the same time as the VM itself rather than building the network ahead of time.

Searched for Virtual Machines, selected Create → Virtual machine, and filled out the Basics:

SettingValueSubscriptionmy subscriptionResource groupaz104-rg5 (created new)Virtual machine nameCoreServicesVMRegion(US) East USAvailability optionsNo infrastructure redundancy requiredSecurity typeStandardImageWindows Server 2025 Datacenter - x64 Gen2SizeStandard_D2s_v3UsernamelocaladminPassword(complex password)Public inbound portsNone

<img width="630" height="766" alt="image" src="https://github.com/user-attachments/assets/29e9a04c-2d0e-42aa-b03d-887776a7da86" />

Left the Disks tab on defaults, then on Networking chose Create new for the virtual network:

SettingValueNameCoreServicesVnetAddress range10.0.0.0/16Subnet NameCoreSubnet address range10.0.0.0/24

On the Monitoring tab, disabled Boot diagnostics — not needed for this lab.

Review + create → Create, then moved straight on to the next task without waiting for it to finish deploying.


Worth noting — the virtual network got created as part of standing up the VM here. You could just as easily build the network infrastructure first and add VMs into it afterward; both approaches work.




Task 2: Create a VM in a Different Virtual Network

Second VM, same idea, but landing in a totally separate network — ManufacturingVnet — so I'd have two isolated networks to test peering between.

Same path: Virtual Machines → Create → Virtual machine, Basics tab:

SettingValueSubscriptionmy subscriptionResource groupaz104-rg5Virtual machine nameManufacturingVMRegion(US) East USSecurity typeStandardAvailability optionsNo infrastructure redundancy requiredImageWindows Server 2025 Datacenter - x64 Gen2SizeStandard_D2s_v3UsernamelocaladminPassword(complex password)Public inbound portsNone

Defaults on the Disks tab, then Create new on the Networking tab for the network:

SettingValueNameManufacturingVnetAddress range172.16.0.0/16Subnet NameManufacturingSubnet address range172.16.0.0/24

Disabled Boot diagnostics on Monitoring again, then Review + create → Create.

[ADD SCREENSHOT HERE]


Task 3: Test Connectivity with Network Watcher

With both VMs deployed and running, the next step was confirming what should happen by default: nothing. Two VMs in two completely separate, unpeered networks shouldn't be able to reach each other yet.

Searched for Network Watcher, opened Connection troubleshoot under Network diagnostic tools, and set up the test:

FieldValueSource typeVirtual machineVirtual machineCoreServicesVMDestination typeSelect a virtual machineVirtual machineManufacturingVMPreferred IP VersionBothProtocolTCPDestination port3389Source port(blank)Diagnostic testsDefaults

[ADD SCREENSHOT HERE]

Ran the test — took a couple minutes for results to come back. Connectivity test came back Unreachable, which tracks, since the two VMs are sitting in completely different virtual networks with nothing connecting them yet.


Task 4: Configure Virtual Network Peering

This is the actual fix — setting up peering between CoreServicesVnet and ManufacturingVnet so resources in each can reach the other.

From CoreServicesVnet, under Settings I opened Peerings and added a new one:

ParameterValuePeering link nameManufacturingVnet-to-CoreServicesVnetVirtual networkManufacturingVnet (az104-rg5)Allow 'ManufacturingVnet' to access 'CoreServicesVnet'Selected (default)Allow 'ManufacturingVnet' to receive forwarded traffic from 'CoreServicesVnet'SelectedPeering link nameCoreServicesVnet-to-ManufacturingVnetAllow 'CoreServicesVnet' to access 'ManufacturingVnet'Selected (default)Allow 'CoreServicesVnet' to receive forwarded traffic from 'ManufacturingVnet'Selected

Clicked Add.

[ADD SCREENSHOT HERE]

Back on CoreServicesVnet's Peerings page, the CoreServicesVnet-to-ManufacturingVnet peering showed up — refreshed the page until the status read Connected. Switched over to ManufacturingVnet and confirmed the matching peering was there too, also showing Connected.

[ADD SCREENSHOT HERE]


Task 5: Retest the Connection with Azure PowerShell

With peering in place, time to confirm the connection actually works now — this time testing from inside the VMs themselves using PowerShell instead of Network Watcher.

First grabbed the private IP of CoreServicesVM from its Overview blade under the Networking section — needed that for the next step.


There's more than one way to test this. Network Watcher works too, or you could RDP directly into the machine and run test-connection from inside. Worth trying RDP at some point just to see it work, but this task uses Run Command instead.



On CoreServicesVM, under Operations → Run command → RunPowerShellScript, I ran:

powershellEnable-NetFirewallRule -DisplayGroup "Remote Desktop"

This opens up the Windows Firewall rule that allows inbound RDP traffic — without it, the connection test below would fail even with peering working correctly.

Once that finished, switched over to ManufacturingVM, same Run command → RunPowerShellScript path, and ran:

powershellTest-NetConnection <CoreServicesVM private IP address> -port 3389

Took a minute or two to run. This time the test succeeded — which makes sense, since peering is now in place and the firewall rule allowing RDP traffic is enabled on the destination VM.

[ADD SCREENSHOT HERE]


Task 6: Create a Custom Route

Last task — adding a perimeter subnet to CoreServicesVnet and setting up a custom route so traffic headed into the core network gets directed through a network virtual appliance (NVA) first, instead of going straight in. (The NVA itself isn't actually deployed here — this just sets up the routing that would point to one.)

On CoreServicesVnet, under Subnets, added a new one:

SettingValueNameperimeterStarting address10.0.1.0/24

Then searched for Route tables, hit + Create:

SettingValueSubscriptionmy subscriptionResource groupaz104-rg5RegionEast USNamert-CoreServicesPropagate gateway routesNo

Review + create → Create.

[ADD SCREENSHOT HERE]

Opened the deployed rt-CoreServices resource, went to Settings → Routes → + Add, and defined a route pointing traffic toward the future NVA:

SettingValueRoute namePerimetertoCoreDestination typeIP AddressesDestination IP addresses10.0.0.0/16 (the CoreServices network)Next hop typeVirtual applianceNext hop address10.0.1.7 (placeholder for a future NVA)

Last step was actually associating this route table with the perimeter subnet — under Subnets → + Associate:

SettingValueVirtual networkCoreServicesVnet (az104-rg5)SubnetPerimeter

[ADD SCREENSHOT HERE]

This sets up a user-defined route that would push traffic from the perimeter/DMZ subnet through the NVA before it ever reaches the rest of the CoreServices network.


Cleanup

If running this on a real subscription, delete the resource group afterward to avoid leftover cost:


Azure portal: select the resource group → Delete the resource group → enter the name to confirm → Delete (confirm a second time)
PowerShell: Remove-AzResourceGroup -Name resourceGroupName
CLI: az group delete --name resourceGroupName



Key Takeaways


By default, resources in different virtual networks can't communicate with each other.
Virtual network peering connects two or more VNets so resources on either side can reach each other.
Peered VNets behave like one network for connectivity purposes once peering is active.
Traffic between peered VMs travels over the Microsoft backbone, not the public internet.
System routes exist automatically for every subnet. User-defined routes can override or extend those defaults — useful for forcing traffic through something like an NVA.
Network Watcher is the go-to toolset for testing, diagnosing, and monitoring Azure network connectivity.
