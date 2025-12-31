# Small-Enterprise-Network-
This document describes the design and implementation of a small enterprise network in a virtualized environment, using Microsoft and Linux technologies to provide centralized authentication, name resolution, and IP address management through Active Directory.

1. Enterprise Layout and Design

The enterprise network consists of the following components:
-  Domain Controller (DC)
-  DNS Server (hosted on the DC)
-  HCP Server (hosted on the DC)
-  Linux Server (domain-joined)
-  Windows Client Workstations (domain-joined)
-  Organizational Unit (OU): Toronto
    -   Two domain users
    -   Users are members of the R&D security group
    -   Group Policies applied to restrict and control services

All systems are hosted as virtual machines using Oracle VirtualBox on an isolated internal network. Infrastructure services are consolidated onto a single Domain Controller to reflect a small-scale enterprise deployment.

2. Server Installation and Role Configuration

2.1 Windows Server Deployment
-  Downloaded Windows Server 2022 (x64) ISO
-  Created a virtual machine to act as the core infrastructure server
-  Disabled non-essential services (e.g., Windows Update) to reduce background overhead
  
2.2 Role Installation
Using Server Manager, the following roles were installed:
-  Active Directory Domain Services (AD DS)
-  DNS Server (installed automatically during DC promotion)
-  DHCP Server
  
2.3 Domain Controller Promotion
The server was promoted to a Domain Controller by creating a new forest:
-  Domain Name: mississaugaDC.com
-  DNS: Installed and integrated with Active Directory
-  DSRM Password: Configured during promotion

After promotion, the server rebooted and became the authoritative Domain Controller for the environment.

<img width="975" height="580" alt="image" src="https://github.com/user-attachments/assets/25797ae5-85bf-4656-aba1-6fdcef2fa2d0" /><br>

3. Network Design and Subnetting

A custom private network was designed based on 5 required hosts. Host Server, two virtual machines, Linux server, and gateway (placeholder). A /27 subnet was selected to support current hosts while limiting how far broadcast traffic is allowed to travel in the network. While reflecting efficient enterprise IP planning.

3.1 Network Configuration
-  Network Address: 10.0.0.0/27
-  Subnet Mask: 255.255.255.224
-  Total IP Addresses: 32
-  Usable Hosts: 30
-  Block Size: 32
-  Broadcast Address: 10.0.0.31
-  Usable Host Range: 10.0.0.1 – 10.0.0.30

This subnet provides sufficient addressing while maintaining efficient IP utilization.

3.2 Static IP Address Assignment
Infrastructure systems were assigned static IP addresses to ensure reliability and prevent IP conflicts.

   `Device	               Role	                 IP Address	        Assignment Type` <br>
   `Server01	             DC / DNS / DHCP	     10.0.0.2/27	         Static` <br>
   `LinuxServer01	         Domain Member Server	 10.0.0.3/27	         Static` <br>
   `Gateway (Logical)	     Placeholder	         10.0.0.1/27	         Static`
  
The gateway address is configured for consistency, although no physical router exists in this isolated lab environment.
The DNS server IP was set to 10.0.0.2, allowing all clients to resolve domain resources correctly.

<img width="975" height="579" alt="image" src="https://github.com/user-attachments/assets/67f6f1a5-caa4-45cf-994e-63ba6354a0c9" />

4. DHCP Configuration

The DHCP server was configured and authorized within Active Directory. 

4.1 DHCP Scope Configuration
-  Scope Range: 10.0.0.10 – 10.0.0.20
-  Subnet Mask: 255.255.255.224
-  Default Gateway: 10.0.0.1
-  DNS Server: 10.0.0.2
  
Static IP addresses (10.0.0.1 – 10.0.0.3) were excluded from the DHCP scope to avoid conflicts and allow future expansion.

<img width="975" height="581" alt="image" src="https://github.com/user-attachments/assets/b132b958-4794-4e43-a29d-b800fc12e2cf" /><br>

4.2 Connectivity and Validation Testing
The following tests were performed from the server and client systems:
-  Pinged local host → Success
-  Pinged Domain Controller (10.0.0.2) → Success
-  Pinged domain name (mississaugaDC.com) → Success
-  Pinged gateway (10.0.0.1) → Failure (expected)
  
Gateway failure is expected due to the absence of a router and confirms the isolated nature of the internal network. However, the gateway address exists logically for configuration consistency and future scalability despite no physical router being present. 

<img width="598" height="686" alt="image" src="https://github.com/user-attachments/assets/a8e34029-67cc-4097-9362-f94dda34b27d" /><br>

5. Client Virtual Machine Deployment

5.1 Client VM 1

Windows client virtual machines were created with the following specifications:
-  Memory: 2048 MB
-  Storage: 50 GB
-  Processors: 2
-  Network Adapter: Host-Only Adapter

5.2 Cloning VM 1 to create VM 2

<img width="975" height="579" alt="image" src="https://github.com/user-attachments/assets/d4b6124c-9bcc-436f-8b46-41e225c8de5e" /><br>
6. Client DHCP Configuration via PowerShell

Client systems were configured to obtain IP and DNS settings automatically using PowerShell:
-  Set-NetIPInterface -InterfaceAlias "Ethernet" -Dhcp Enabled
-  Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ResetServerAddresses
-  ipconfig /release
-  ipconfig /renew

6.1 DHCP Verification Client 1

Client 1 successfully received IP configuration from the DHCP server:
-  IPv4 Address: 10.0.0.12
-  Subnet Mask: 255.255.255.224
-  Default Gateway: 10.0.0.1
-  DNS Server: 10.0.0.2
-  DHCP Server: 10.0.0.2
  
<img width="924" height="739" alt="image" src="https://github.com/user-attachments/assets/311bbf88-e65f-4e83-8000-6b5ec4b73a52" /><br>

6.2 DHCP Verification Client 2

Client 2 successfully received IP configuration from the DHCP server:
-  IPv4 Address: 10.0.0.13
-  Subnet Mask: 255.255.255.224
-  Default Gateway: 10.0.0.1
-  DNS Server: 10.0.0.2
-  DHCP Server: 10.0.0.2

<img width="875" height="793" alt="image" src="https://github.com/user-attachments/assets/8b550326-7b7d-4a8f-adde-fc5a9d783ed0" /><br>

7. Domain Join Process

Once connectivity and DNS resolution were confirmed, the clients were joined to the domain using PowerShell:
-  Add-Computer -DomainName mississaugaDC.com -Credential mississaugaDC\Administrator -Restart

7.1 Client 1 Domain Membership Verification

After reboot, domain membership was verified:
-  Get-ComputerInfo | Select CsDomain, CsDomainRole

Result: 
-  Domain: mississaugaDC.com
-  Role: MemberWorkstation

<img width="975" height="540" alt="image" src="https://github.com/user-attachments/assets/35d55bfe-131a-4ca9-869d-310dcacb081e" /><br>

7.2 Client 2 Domain Membership Verification 

<img width="975" height="632" alt="image" src="https://github.com/user-attachments/assets/bd89e75e-a7cc-4f72-8165-8c0ef1165672" /><br>

8. Creating and configuring Ubuntu Server to join domain

8.1 Installing Network Manager and Configuring Adapters 

-	Downloaded and enabled NetworkManager
-	Applied NIC account to device (i.e. enp0s8) to manage it from the Network Manager. 
-	Disabled ipv6 on both NIC adapters. And set the priority of the enp0s8 as the main DNS client. So, the systemd knew where to resolve DNS queries first. 
-	Modified the enp0s8 to receive DHCP automatically.
-	The domain controller was added to the /etc/hosts file to ensure name resolution consistency during early domain-join and Kerberos initialization.
-	Set the NIC as “up” so it can resend a broadcast to retrieve an IP address from the DHCP server on its network. It was successful and received the below address.

Ubuntu Server successfully received Ip configuration from the DHCP server:
-  IPv4 Address: 10.0.0.18/27
-  Subnet Mask: 255.255.255.224
-  DNS Server: 10.0.0.2
-  DHCP Server: 10.0.0.2

<img width="975" height="508" alt="image" src="https://github.com/user-attachments/assets/70cacf57-9ea3-409c-94d1-e790f75dc05a" />
<img width="979" height="393" alt="image" src="https://github.com/user-attachments/assets/2c84b423-c3f2-445d-8275-e894cbeb16cd" /><br>

Checks: 

-  Pinged: mississaugaDC.com = success
-  Pinged: 10.0.0.2 (DNS, DHCP, Host server, Ip address) = Success

8.2 Linux-Active Directory Integration Using Kerberos and SSSD

To join my Ubuntu server to my domain the ubuntu server must be able to communicate with the DC and recognize the domain.  Here are the following steps used to achieve this: 

Step 1: Installing the "Bridge" Software

Installed realmd, SSSD, and adclI & Kerberos. This will help the Ubuntu server authenticate with the DC using the software used by the DC. Below are what they do to help this joining process:
-  Realmd: The manager that automates the join.
-  SSSD (System Security Services Daemon): This is the "brain." It stays running in the background, talking to the DC and caching credentials so you can log in even if the DC is briefly offline.
-  Adcli & Kerberos: The tools that handle the actual encrypted "handshake" and account creation in the Windows database.

Step 2: The Handshake (realm join)
Joining the domain mississaugaDC.com
-  Sudo discover mississaugaDC.com: Searches the network for domain and lists required software to authenticate with the domain.
-  Sudo join mississaugaDC.com: Join the domain, but requires the administrator’s password to gain access.
-  What happened: When you ran realm join, the Ubuntu server reached out to the DC, proved it had the Administrator password, and asked to be added.
-  The Result: The DC created a Computer Object in its database and shared a secret key with the Linux server.

Result:<br>
<img width="486" height="465" alt="image" src="https://github.com/user-attachments/assets/62b78517-4e44-4c82-ad4c-44ef7bac1017" />

Step 3: Identity and Permission Mapping
Once joined, the server knew who the users were, but it didn't know what they were allowed to do.

New user home directory creation: 
-	pam-auth-update was used to enable mkhomedir. This ensures that when Administrator@mississaugaDC.com logs in, Linux automatically builds them a folder at /home/administrator.
Setting administrative permissions within the Linux server (Sudo Rights)
-	 I edited the /etc/sudoers; to tell the linux server "If someone is a Domain Admin in Windows, let them have root powers on this Linux machine."

9. Task Automation in Windows

9.1 Script to Automate ADUser Creation 

-	The script below asks the administrator for username, SamAccountName, email-address, sets the account to “change password at login”, and more. Once the user account has been created, the Get-ADuser cmdlet checks to see if the user account has been created for validation purposes.

<img width="975" height="580" alt="image" src="https://github.com/user-attachments/assets/60f45ceb-0673-4bec-bbc5-e407765be820" /><br>

9.2 Script to Automate ADGroup Creation 

-	The script below will request the administrator for GroupName, SamAccountName, Category, Scope, and more. Once this information is received the script will utilize the New-ADGroup cmlet with variables used as inputs for each parameter required. And, displays the entry using Get-ADGroup.

<img width="975" height="579" alt="image" src="https://github.com/user-attachments/assets/a25e75a4-6146-44c8-9e17-be5ccae15b80" /><br>

9.3 Script to Automate ADOrganizationalUnit Creation 

-	The script utilizes name, and path variables that store the administrators input. These inputs are then used to create the New-ADOrganizationalUnit. For validation checks, the script then displays the newly created Organizational Unit. 

<img width="975" height="579" alt="image" src="https://github.com/user-attachments/assets/7f597fd9-e173-4cf5-a2c4-c5b9dd6b0ab9" /><br>

9.4 Script to Automate Adding Users to Groups 

-  The script stores the group identity specified by the administrator in a variable and then prompts for the username of the user to be added to the selected group. To allow multiple entries, the script uses a while loop. The loop first checks whether the administrator has entered “done”; if true, the loop exits. If not, it checks whether the input is blank—if so, the loop continues and prompts again. If both checks return false, the script verifies whether the user exists. If the user exists, they are added to the specified group; otherwise, the else condition returns a “user doesn’t exist” error message, indicating that the user must be created before being added to the group.

<img width="975" height="578" alt="image" src="https://github.com/user-attachments/assets/970901b5-91d6-4fea-acad-614493336e23" /><br>

Conclusion
<br>
<br>
This project successfully demonstrates the deployment of a small enterprise network using Active Directory, DNS, DHCP, and both Windows and Linux clients. The environment reflects real-world enterprise practices, including static IP allocation for infrastructure, dynamic addressing for clients, centralized authentication, and structured policy management. The network is scalable, secure, and suitable for further experimentation with Group Policy, access control, and cross-platform authentication. Furthermore, it utilizes scripting to automate simple tasks for the administrator within the environment. 
























