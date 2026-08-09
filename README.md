<h1>Active Directory Project</h1>

<h2>Description</h2>
I built this project to demonstrate the deployment, configuration, and automation of an enterprise-modeled Active Directory infrastructure within a fully isolated virtual environment. Using Windows Server 2022 and Windows 11, I established a fully operational Domain Controller managing network routing via RAS/NAT, automated IP addressing through custom DHCP scopes, and centralized identity management. To optimize administrative workflows, I created custom PowerShell scripts to automate bulk user creation and provide real-time, interactive account auditing and incident remediation.

<br />
<br />
<img width="651" height="701" alt="FinalDia drawio" src="https://github.com/user-attachments/assets/69aafc25-3408-4523-8e02-39a64da012aa" />

<br />
<br />


<h2>Languages and Utilities Used</h2>

- <b>PowerShell</b> 
- <b>Active Directory Module</b>

<h2>Environments Used </h2>

- <b>Windows2022Server</b>
- <b>Windows 11</b> (25H2)
<h2>Program walk-through:</h2>

<p align="left">
<b>Lab Specifications & Network Architecture</b>
 
This lab project uses Windows Server 2022 as the Domain Controller and Windows 11 Pro (25H2) as the client machine. The resources allocated for both virtual machines are 4 CPU Cores, 4GB of RAM, and 80GB of storage. The Domain Controller is configured with both a NAT and an Internal NIC, while the Client is configured with only an Internal NIC. <br/>

<img width="990" height="870" alt="image" src="https://github.com/user-attachments/assets/bff4b740-7e79-4b5e-afb9-72a40ebdff05" />
<p align="left">
<b>Installation & Configuration Details</b>

I used Windows Server Desktop Experience (GUI) to provide clear visual confirmation of configurations that I was going to be implementing.

<b>Steps taken during and immediately after the setup process:</b>
- <b>Credential Standardization:</b> For the scope of this lab, I used one complex password across both the domain controller and client machine to simplify deployment of the VMs while still meeting the default Windows Server password complexity requirements.
- <b>VirtualBox Guest Additions:</b> Then I installed VirtualBox Guest Additions on both virtual machines immediately following the setup process. This step is important as it improves mouse pointer integration, display scaling, and overall system responsiveness.
<br/>
<br />
<h2></h2>
<b>Network Interface & DNS Settings</b>

I set the Domain Controller with a static IPv4 address on the internal network. The default gateway was left blank because leaving it populated would cause a conflict inside the Domain Controller, and the DC itself was configured to provide the default gateway through NAT.

<b>DNS Loopback Integration</b>

The Preferred DNS Server was set to the loopback address 127.0.0.1, so the Domain Controller resolves names using its own local DNS service. Since Active Directory heavily relies on DNS to map the location of domain resources, this will allow Active Directory to reliably use name resolution within the lab environment.

<img width="1024" height="768" alt="network settings ipv4" src="https://github.com/user-attachments/assets/606429f1-1948-47e8-a821-0aebbf2f2da3" />
<p align="left">
<b>Computer Renaming & System Identity</b>
 
Before configuring roles or promoting the server, I changed the computer name from the default Windows-generated string to a standardized hostname: 2022-ServerDC. This is considered best practice because the hostname is used throughout Active Directory and related services to identify the server. Renaming a Domain Controller after promotion can introduce complications and require additional steps to update directory records and service references.
<br />
<br />
<br />
<h2></h2>
<b>Active Directory Domain Services (AD DS) Installation</b>

Next, I installed AD DS, which sets up the core files needed to promote the server to a Domain Controller.
- <b>Active Directory Domain Services:</b> I installed this service because it is the foundation behind centralized infrastructure for user authentication, network resource management, and secure access to the domain.
- <b>Group Policy Management:</b> I installed this to allow baseline configurations to be pushed later on the lab, and it is used to push software updates and restrict access to assets inside the domain.
- <b>Remote Server Administration Tools (RSAT):</b> Installed for GUI and PowerShell tools that allow the configuration management of roles.

The last step was to promote the server to a Domain Controller by selecting "Add a new forest" and naming the root domain mydomain.org.

<img width="1024" height="768" alt="AD installation" src="https://github.com/user-attachments/assets/61587d6d-14a8-4a0a-9bed-116e5aa84ca4" />
<b>Administrative Account Separation:</b> Following the initial domain setup, I followed the enterprise best practice which dictates that configuring and installing other Active Directory services should be done with an actual Administrative account rather than a default built-in Windows Administrator profile. To execute this, I used Active Directory Users and Computers or ADUC to create a new Organizational Unit (OU) named "_ADMINS". Then, I created a new user account inside this OU and added it to the Domain Admins group.

<br />
<br />
<h2></h2>
<b>Routing and Remote Access (RAS/NAT)</b>

Following the login to the new Administrator account, I deployed the Routing and Remote Access Service (RRAS). Configuring RRAS with Network Address Translation (NAT) allows internal client virtual machines to securely access the internet through the host server's external network interface. This configuration allows the server to act as a gateway for the client machines while isolating them from the physical home network.

<img width="1024" height="767" alt="routing and remote" src="https://github.com/user-attachments/assets/52bef14d-35a8-4dd4-b51a-0e7342d6f5dc" />

<br />
<br />
<h2></h2>
<b>DHCP Server Installation</b>

In this phase, I installed the DHCP Server role, which automates network configuration for the internal virtual machine clients. This allows the internal clients to automatically receive a valid IP address, subnet mask, default gateway, and the correct DNS settings.

<img width="1024" height="767" alt="DHCP" src="https://github.com/user-attachments/assets/1b3429e7-d5e6-4c14-9deb-dc01a8b10a16" />

<br />
<br />
<h2></h2>
<b>DHCP Scope</b>

To complete the DHCP role deployment, I configured the new DHCP scope within the 172.16.0.0/24 subnet, creating an address pool from 172.16.0.100 to .200. Within the scope options, I set the default gateway to 172.16.0.1, ensuring all internal clients automatically route through the network's local NAT gateway to access external networks. 

<img width="1024" height="767" alt="DHCPFinal" src="https://github.com/user-attachments/assets/e7e2729e-f457-4a04-801f-9d179c610671" />

<br />
<br />
<h2></h2>
<b>Generate Unique Names</b>

Instead of downloading a generic list of names from the internet, I created a custom PowerShell script to generate a completely randomized list of 500 unique names. The script pulls from arrays of first and last names, dynamically checks each combination against a high-efficiency hashtable to ensure uniqueness, and skips duplicates. Once 500 unique names are successfully generated, they are exported to a text file "names.txt", which will serve as the data input for the bulk provision script.

<img width="1024" height="767" alt="realtxt" src="https://github.com/user-attachments/assets/6ff4d43e-ae0d-4084-a877-b2edb631bbc3" />
<br />
<br />
<br />
<h2></h2>
<b>Bulk Active Directory Provisioning</b>

With the raw data generated, I created a second custom PowerShell script to parse "names.txt" and automatically create the accounts. The script reads each line, splits the full name into individual first and last name strings, and dynamically constructs standardized `firstname.lastname` usernames. Using a parameter splatting hashtable "@UserParams", the script maps out the unique user attributes-including a standardized initial password and different Employee IDs-before executing `New-ADUser` to rapidly create all 500 records directly to an organizational unit named "_EMPLOYEES".

<img width="1024" height="767" alt="realcreate" src="https://github.com/user-attachments/assets/7044ac29-1236-4c22-a19e-04a9211e3c3b" />
<br />
<br />
<br />
<h2></h2>
<b>Infrastructure and Network Connectivity Validation</b>

Directly following the execution of the bulk script, I booted the Windows 11 client machine and accessed the local user account to perform a pre-join audit. Using the Command Prompt, I performed `ipconfig` to verify that proper IP network addressing was within the isolated lab environment. I ran two connectivity tests using ping: an internal ping which validated DNS resolution and low-latency traversal to the Active Directory Domain Controller (mydomain.org), and an outbound ping to an external network (google.com), confirming functional NAT routing and gateway access.

<img width="1024" height="768" alt="realping" src="https://github.com/user-attachments/assets/0fcf728c-16f4-4712-9254-5a3ed35771fc" />

<br />
<br />
<br />
<h2></h2>
<b>Active Directory Domain Integration</b>

After validating network connectivity and DNS resolution, I joined the Windows 11 Pro machine (client1) to the "mydomain.org" Active Directory domain using privileged credentials that I created back in the "_ADMINS" OU. This established secure communication between the client and the Domain Controller, completing the domain join.

<img width="1024" height="768" alt="realjoin" src="https://github.com/user-attachments/assets/52c296db-f309-49d0-91b2-59f43d93389d" />
<br />
<br />
<br />
<h2></h2>
<b>User Authentication and Domain Session Verification</b>

Following the domain join and a system restart, to verify that the domain was successfully integrated, I logged into the client machine with the domain account "andrew.jones", which was provisioned during the bulk script. I used the Command Prompt to run the `whoami` utility, confirming an active domain session ("mydomain\andrew.jones"). This test validated that the workstation securely communicates with the Domain Controller and that script-created accounts are fully operational for enterprise network access.

<img width="1024" height="768" alt="whoami" src="https://github.com/user-attachments/assets/3e8aff1d-61e3-4f84-bd3c-fdd8864a72ff" />
<br />
<br />
<br />
<h2></h2>
<b>Group Policy Enforcement and Security Baseline Verification</b>

I created a security baseline configuration using the Group Policy Management Console (GPMC). To deliver these security thresholds to the client machine, I ran a forced policy update. Using the Command Prompt on the target machine, I ran the `gpresult /r` utility to audit the configuration. As shown in the picture, the result verifies that the "Sec_User_Baseline" Group Policy Object is actively applied to the user account within the "_HR" Organizational Unit, successfully enforcing the baseline security settings.

<img width="1024" height="768" alt="finalbaseline" src="https://github.com/user-attachments/assets/78661b73-1cd2-4d74-97bc-d04983b5cf9b" />
<br />
<br />
<br />
<h2></h2>
<b>Active Directory Security Baseline & Automated Incident Response Lab</b>

To test the interactive automation script under a realistic scenario, I changed the domain's Account Lockout Policy baseline temporarily to enforce a strict lockout threshold of 2 invalid logon attempts.
- <b>State Detection:</b> I intentionally locked the user account "andrew.perez" via a Windows 11 workstation (shown on the right virtual machine). I ran the remediation script on the Domain Controller (shown on the left virtual machine) which queried Active Directory and flagged the account's boolean state with an automated Red alert (Locked Out: True).
- <b>Remediation and Compliance:</b> I selected Option "1" to instantly unlock the account's lockout status in Active Directory, updating the terminal status to a compliant Green (Locked Out: False). Subsequently, I chose option 2 to reset the account's credential to a temporary default (Password0).
- <b>Security Enforcement:</b> Because the script natively utilizes the `-ChangePasswordAtLogon $true` parameter within the `Set-ADUser` cmdlet, it enforces strict credential hygiene. Clicking "OK" on the client workstation triggered a mandatory corporate password change dialog before allowing network authentication.

<img width="2560" height="1440" alt="Screenshot 2026-06-28 193631" src="https://github.com/user-attachments/assets/668a9d37-cecf-4cc7-9fda-cf4a8e2c5745" />

<img width="2560" height="1440" alt="Screenshot 2026-06-28 193748" src="https://github.com/user-attachments/assets/ec6f79c1-518a-43a3-bead-b1ace455ac00" />

</p>

<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
