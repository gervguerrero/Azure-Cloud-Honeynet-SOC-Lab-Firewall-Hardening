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
![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/4b469de6-1eab-4cd1-9c6b-1143fc741636)


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

 <br/> 

Here we can see evidence without this security control, that any external IP can perform RDP into our Windows 10 virtual machine, as well as ping the machine:
![VMfw2](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/4900fc97-dc22-440d-9e8a-8ead177401a3)
![VMfw3](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/85e5faf5-c222-4fee-a005-7cbab47ddb32)

<br/> 

## Configuring Controls (NSG for VMs)

Just like the previous Network Service Group, we can change the settings to only accept inbound traffic from 1 specific IP address:
![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/a56b3641-ed44-4922-b2f9-2e313af902fe)

<br/> 

## Threat Actor Access After Controls (NSG for VMs)
After implementing these NGS for VMs controls, we can see that a threat actor's previous access to the Windows 10 VM has been blocked, effectively eliminating the RDP brute force seen previously:

![VMfw5](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/deec2a0c-af13-462a-840a-0a106ee6531f)
![VMfw6](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/4b014f2c-4fad-4553-9021-48d043251231)

<br/> 

Here is what the network looks like blocking unauthorized traffic from the internet using a Network Service Group for Virtual Machines:

![Secured Network VM NSG](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/25a65c53-ee46-4ec6-837c-95723f1cdf7d)

<br/> 
<br/> 

# 4. Operating System Firewalls and Blob Storage/Key Vault Firewalls
## Threat Actor Access Before Controls (OS Firewalls/Resource Firewalls)
In the Operating System itself for each virtual machine, there are usually firewall applications that are native and can be activated to block any unwanted traffic. 

**Windows 10:** Windows Firewall

**Ubuntu:** UFW Firewall

Meanwhile, both the Blob Storage and Key Vault have firewall/network settings that can be changed in the Azure portal that disable public access to these resources. 

Here is the open Windows 10 Firewall that allows any inbound traffic from the public internet that allows attackers to brute force our Windows machine:

![OSfw0](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/d0d81d37-1f7f-4e0e-be17-bf8522d2e05b)

<br/> 

Here we can see evidence without this security control, that any external IP can perform RDP into our Windows 10 virtual machine, as well as ping the machine:

![OSfw1](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/947e2622-f873-4842-9b36-7443a6ed439c)
![OSfw2](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/8a5079bc-f34d-4c30-b6e9-06764161f25f)


<br/> 

Here is the Ubuntu Firewall that is inactive and not configured that allows anyone to SSH in:

<img width="268" alt="Ubuntufw1" src="https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/6aba298a-e3fc-49ec-831b-064e51b3b724">

<br/> 
<br/> 

Here we can see evidence without this security control, that any external IP can perform SSH into our Ubuntu virtual machine:
![tempsnip](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/09f3bd49-9a59-411b-8dd3-9387022426e3)

<br/>
<br/>

Here is the Blob Storage Firewall/Networking rule that allows public acccess: (Keyvault has same setting) 

![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/0815c9cb-100f-4bcc-a556-8e8d27f4d38a)

<br/>

## Configuring Controls (OS Firewalls/Resource Firewalls)

This is hardening the Windows 10 Firewall to only accept remote inbound connections from 1 specific IP:

![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/ec9c2425-21d9-439d-beef-4b889030fb5a)
![OSfw3](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/c52a9d5d-a5c3-48ac-983c-48d31f1c360a)

<br/>

This is hardening the Ubuntu UFW Firewall to only accept remote inbound connections from 1 specific IP:

<img width="381" alt="Ubuntufw2" src="https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/0faf076f-7da1-46d2-b594-1debeae3400b">

![tempsnip2](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/b958cbdd-8770-4073-982d-09447bf318e0)

![tempsnip3](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/4cd4c258-a008-40e3-8e07-e3cb5bbb4855)

<br/>

This is hardening the Blob Storage/Key Vault access by disabling public access from the the internet: **(Note that this is to be combined with Private Endpoints which will be discussed next.)**

<img width="702" alt="OSfw6" src="https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/715b86f9-afc8-4f70-b255-0c548a77e227">

<br/>

## Threat Actor Access After Controls (OS Firewalls/Resource Firewalls)

After implementing these OS Firewall/Resource Firewall controls, we can see that a threat actor's previous access to the Windows 10 VM has been blocked, effectively eliminating the RDP brute force seen previously:

![OSfw4](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/2ac554ad-11f2-439c-8f74-19085f9f8728)

![OSfw5](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/2b62ae10-9998-4c80-919b-d7fb15a6f517)

<br/>

The same access control has been applied to the Ubuntu VM:

![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/049d769a-703e-4ce5-aba3-945b0195ecfc)

<br/>

Here is what the network looks like blocking unauthorized traffic from the internet using OS Firewalls/Resource Firewalls: 

![Secured Network Asset Firewall](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/d279b32a-47b4-492b-986d-468c671ec605)

<br/> 
<br/> 

# 5. Private Endpoint Protection for Blob Storage/Key Vault
## Threat Actor Access Before Controls (Private Endpoints)

Without any configuration for Private Endpoints for our Azure resources (Blob Storage/Key Vault), any threat actor who would be able to view the resources from any public IP space. 

Allowing public access:

![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/94d2c693-56f7-4537-ac1c-cc8b616e667c)

No Private Endpoint Configured:

![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/33462a6b-9f44-4af6-b766-98069bb401b2)

Here is what the threat actor's access to the resources looks like from an IP outside of the Azure network. They can view the Key vault Secrets and see the sensitive passwords:

![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/3d514ce5-6209-49cd-90ac-6db376ee8290)

<br/> 

 ## Configuring Controls (Private Endpoints)

 Here we disable Public Access for the Key Vault:
 
 ![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/391220f8-1764-4d24-9461-e3561098c909)

Here we configure a Private Endpoint to only allow access to the resources from INSIDE the Azure network:

![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/d9e447ec-ba77-4c19-86a5-107377f7d73e)

![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/b0a81513-871f-47fd-b694-61f4bafa6a20)

![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/6baac2de-42fc-4ad3-8bbf-4384f000d90b)

![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/02632b7b-9393-485c-bde5-b86781df1e51)

<br/>

 ## Threat Actor Access After Controls (Private Endpoints)

Here is a threat actor's access to the Key Vault after enabling these controls. Note that the nslookup shown resolves to a Public IP address only. The Private Endpoint control only allows access from a Private IP inside the Azure network, which is why Azure states Public access is disabled:

![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/72e7b254-721c-4c7a-9190-482250ae8ab4)

This is what the Private Endpoint access looks like from INSIDE the Azure network. Note the Private IP returned in the nslookup:

![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/3b008ee1-33c8-45f3-af76-2fcc8efac227)


Here is the threat actor's access to the Blob Storage after enabling the same controls:

![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/c8d2b968-0193-4fa9-8617-585a9cfb05bd)

Here is the Private Endpoint access from inside the Azure network. Note the Private IP returned in the nslookup:

![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/294baa8a-d749-4560-bb7e-801341b69370)

<br/>

Disabling public access and creativing Private Endpoints results in a much more secure Azure resources:

![Secured Network Asset Firewall](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/39b7b720-df6a-4f0d-aeee-82349c1d8657)

<br/>
<br/>

# Conclusion

![Secured Network Azure firewall](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/e3943ae0-3887-4f3e-8b6c-041a293a2b5e)

Here we have demonstrated 5 layers of firewall protection in a Microsoft Azure Cloud environmnet following NIST's 800-53 R5 SC-7 Boundary Protection guidelines, effectively hardening the Azure network to prevent brute force attempts from happening and disabling public access of Azure Storage and Azure Key Vault assets. We've shown how each control configured alone can still prevent unwanted access. Using all 5 of these controls together bolster the network security posture and prevent the brute force incident we explored earlier.

**To view the brute force incident that led to hardening the network, click here:**

[Azure-Cloud-Honeynet-SOC-Lab-Incident-Response-Investigation](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Incident-Response-Investigation)

**To view the results of introducing these security controls and the overall Honeynet/SOC lab, click here:** 

[Azure-Cloud-Honeynet-SOC-Lab](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab)

