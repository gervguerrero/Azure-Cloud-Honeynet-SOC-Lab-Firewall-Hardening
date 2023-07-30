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

# Configuring Azure Firewall 

Starting with the most external security control, introducing and configuring an Azure Firewall properly can prevent unwanted remote deskop protocol (RDP) requests to the Azure network.

Here we can see evidence that any external IP can perform RDP into our Windows 10 virtual machine:
![image](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening/assets/140366635/a69ca949-e7d4-4d8b-918b-99d6c8884b64)

