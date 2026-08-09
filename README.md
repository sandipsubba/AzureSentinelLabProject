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
