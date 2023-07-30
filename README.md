# 🔥🔒 Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening 🔒🔥

This page showcases hardening Microsoft Azure Firewalls in an Incident Response process. This builds off my repository [Azure-Cloud-Honeynet-SOC-Lab](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab/tree/main), and [Azure-Cloud-SOC-Lab-Incident-Response-Investigation](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Incident-Response-Investigation/edit/main/README.md)

![final map](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/c1ef655b-4ebf-4b86-b7d7-3060246645a6)


In Microsoft Azure, I built a public facing Honeynet to attract real world cyber attackers to monitor their TTPs. Here I showcase a basic security response action plan to a successful brute force login incident seen in Microsoft Sentinel. This is where the Containment, Eradication, & Recovery phase of the Incident Response lifecycle based on NIST 800-61 takes place. The goal is to successfully implement 5 layers of firewalls based on NIST's 800-53 R5 SC-7 Boundary Protection guidelines, effectively hardening the Azure network to prevent brute force attempts from happening. 

 <br/> 

 # Vulnerable Network 
![uncsecure net map](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/97549972-f13c-445c-9719-38f30ecf44ae)

The Azure cloud environment was left wide open without any real protection from cyber threat actors on the internet. The map shows  5 layers of defenses that were left open for threat actors to directly connect to the virtual machines and brute force/login into.

1. Azure Firewall
2. Network Service Group (NSG) covering the entire virtual network
3. Network Service Group specific to each virtual machine
4. Operating System specific firewalls to each virtual machine and firewall rules specific to the Blob Storage/Key Vault
5. Private Endpoint Protection for Blob Storage/Key Vault (Not configured, not shown on map.)

This page will demonstrate configuring each security control and how it can prevent the brute force incident from occuring. 

 <br/> 

# 1. Azure Firewall 
## Threat Actor Access Before Controls (Azure Firewall)
Starting with the most external security control, introducing and configuring an Azure Firewall properly can prevent unwanted remote deskop protocol (RDP) requests to the Azure network.

Here we can see evidence without this security control, that any external IP can perform RDP into our Windows 10 virtual machine, as well as ping the machine:

![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/a69ca949-e7d4-4d8b-918b-99d6c8884b64)
![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/2f948b45-26d5-448c-a78d-08c0e92d532c)

## Configuring Controls (Azure Firewall)

When introducing an Azure Firewall to the network, a default route needs to be made to route the internal traffic through the firewall to be filtered and inspected. 

First we create a route table:

![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/095374c0-000f-45ba-af04-715d2edfdfb2)
![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/df7b958b-fffd-49ea-b5e3-f9ee019724ce)
 
 <br/> 

Next we have to associate the route table to the virtual subnet our target vm is in. (Windows 10 VM)

In larger environments, we’ll have to attach the route table to any subnet that has clients that needs to access the internet through the firewall. 

This is us associating the subnet our target vm (or client workstations) are in:
![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/54b254c8-9b5c-47c9-85a7-36d975b97ba1)

 <br/> 

Next is setting a default route of 0.0.0.0/0 and setting next hop address of the PRIVATE IP of our firewall.
![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/fb54d557-6b70-42d3-8746-71cfbb2ee22b)

 <br/> 

This default route of 0.0.0.0/0 tells the virtual network to send any traffic that it doesn’t have a route for to the firewall:
![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/fe99795a-33d3-43f2-8b8d-a8e573d95579)

 <br/> 

Next is configuring DNAT on the Azure firewall. (Destination Network Address Translation) DNAT translates the destination IP address and/or port of incoming traffic before it reaches the destination of the requested resources in the Azure virtual network. 
![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/44074ea1-6d3a-4c31-b967-2ab46a6939a0)

<br/> 

Next we are setting some Application rules around HTTP and HTTPS to only allow a destination website of www.disa.mil:
![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/c54f4ef8-279c-4bfd-906e-904c8de0891a)

 <br/> 

Next we are setting some Network rules to allow RDP login from only 1 authorized public IP address. This setting alone denies anyone else and any other network protocols:
![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/27f6e56b-e6e6-4947-96b8-e577102dfeb0)

 <br/> 

Forgot to capture this earlier, but DNS rules were specified in the Network rules to allow DNS queries outbound for authorized websites.
![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/0e287d2e-6d85-4a63-a382-fcfb9db5e06b)

 <br/> 
 <br/> 
 
## Threat Actor Access After Controls (Azure Firewall)
After implementing these Azure Firewall controls, we can see that a threat actor's previous access to the Windows 10 VM has been blocked, effectively eliminating the RDP brute force seen previously:

![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/4448f064-c146-42eb-b6e6-5cb4e7a28381)
![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/4f8084cb-9025-472b-af40-a87a681d9f9d)

Note that the above IP of 20.213.166.111 is now blocked behind the firewall. The the new way to access the Windows 10 VM with installing an Azure Firewall with DNAT is the public facing IP of the firewall, 20.5.100.12.

 <br/> 

Due to our specified rules in the firewall, any external IP besides our 1 authorized IP is blocked:
![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/172f42be-9ac5-47ef-a604-9eedb0b126fd)

 <br/> 

Here we can see a successful RDP login from the 1 authorized IP:
![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/4340a456-5a83-4a34-b7d6-f4918088a540)

 <br/> 

Note that the OS firewall for this Windows 10 VM is disabled, demonstrating that the Azure Firewall configured is what blocks any other external connection:
![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/4850eead-785d-4353-8584-2b92fd111881)

 <br/> 

Here is a demonstration that our firewall HTTP/HTTPS Application rules work. We see that the Windows 10 machine is not able to access www.youtube.com, but is able to access www.disa.mil:
![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/5e6eef92-9025-49d1-ba09-68db7469d484)
![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/4369bd0c-874e-4b89-bec2-e59c82c5d6a5)

 <br/> 

Here is what the network looks like blocking unauthorized traffic from the internet using an Azure Firewall:

![Secured Network Azure firewall](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/bebfd618-487a-4285-b740-ff178deecec4)

Whether the other 4 security controls are operational or not, the Azure Firewall alone will block out any unwanted inbound traffic. (Though they should all be on to practice defense in depth or layered protection.) 

 <br/> 
 <br/> 
 
# 2. Network Service Group (NSG) for Whole Virtual Network

## Threat Actor Access Before Controls (NSG for VNET)
Moving internal to the Azure network, configuring a Network Service Group (NSG) for the entire virtual network can prevent unwanted remote deskop protocol (RDP) requests. Setting the inbound and outbound rules is similar, but much more simple than the Azure Firewall.  

Here is the NSG that allows any inbound traffic from the public internet that allows attackers to brute force our Windows machine:
![Subnetfw1](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/4e771551-fce4-4b89-a37a-b36f73c11071)

 <br/> 

Here we can see evidence without this security control, that any external IP can perform RDP into our Windows 10 virtual machine, as well as ping the machine:

![Subnetfw2](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/7a8b4459-4283-4de3-9f46-9810e44b3dbf)
![Subnetfw3](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/29d277a9-3690-4755-855f-da1b21d7943c)

 <br/> 

 ## Configuring Controls (NSG for VNET)

Since Network Service Groups behave similar to firewalls, we can change the settings to only accept inbound traffic from 1 specific IP address:
![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/0016fc4f-ca4c-4caa-85dd-144730e74368)

 <br/> 

## Threat Actor Access After Controls (NSG for VNET)
After implementing these NGS for VNET controls, we can see that a threat actor's previous access to the Windows 10 VM has been blocked, effectively eliminating the RDP brute force seen previously:

![Subnetfw5](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/931b1c91-5f89-424c-83d9-9571e7eba2b6)
![Subnetfw6](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/da3b95fe-8eb5-4030-af82-6f1d7844c1f1)

 <br/> 

Here is what the network looks like blocking unauthorized traffic from the internet using a Network Service Group for the entire Virtual Network:
![Secured Network Vnet NSG](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/61d5ab64-04ab-4453-89c8-82f14f7c0c9a)



 <br/> 
 <br/> 

# 3. Network Service Group (NSG) for Virtual Machines 
## Threat Actor Access Before Controls (NSG for VMs)
The next internal security control is also a Network Service Group, but it is specific to each Virtual Machine in the Azure network. Each one can be configured separately to accept or deny traffic. In this example we show the NSG for the Windows 10 VM.

Here is the VM NSG that allows any inbound traffic from the public internet that allows attackers to brute force our Windows machine:
![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/3483826e-d855-458f-b481-7c50a612d0fe)
