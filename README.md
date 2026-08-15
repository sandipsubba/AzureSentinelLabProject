<h1>Security Information and Event Management Project</h1>

<h2>Description</h2>
This project demonstrates end-to-end Security Information and Event Management (SIEM) workflows using Microsoft Sentinel, covering threat detection, geographic mapping, and incident containment. The lab environment consisted of a Windows honeypot machine joined to an Active Directory domain controller to replicate real-world enterprise infrastructure. To simulate an attack, I used a remote Ubuntu VM running Hydra to launch an RDP brute-force attack, generating multiple Event ID 4625 (failed logon) entries. Sentinel ingested the telemetry, geolocated the attacker’s IP to Canada East, and triggered an analytics alert on the target account. Once the incident was confirmed, I contained the threat by adding a Network Security Group (NSG) rule to block the attacker’s IP address.

<br />
<br />
<img width="842" height="792" alt="DiagramFinale drawio" src="https://github.com/user-attachments/assets/fb881362-211c-430c-92e3-56efdd4f140b" />

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
 
### Windows Honeypot/Client Deployment
 
<b>Resource Group:</b> The first part of setting up the environment was creating a resource group (`SIEM-LAB-SOC`) in East US 2 to organize and manage all of our related VMs and applications together in one place. <br/>
<b>Virtual Network:</b> The next stage was creating a virtual network (`VNET-SIEM-SOC`) in East US 2 and using the `SIEM-LAB-SOC` resource group so that our VMs could access the internet and connect with each other. <br/>
<b>Virtual Machine:</b> Directly following, the next step was provisioning our honeypot/client machine (`Corporate-US-Net`) running Windows 11 Pro to simulate real-world experience and attract attackers. <br/>

<img width="2560" height="1440" alt="1st" src="https://github.com/user-attachments/assets/5db23687-ac53-4c7e-80c3-8a7c8bd5dae6" />
<p align="left">
<br />
<h2></h2>

### Network Security Group

<b>Inbound Security Rules:</b> Following the deployment of the virtual machine, the next step was to open it to the internet, allowing all traffic through the NSG to the virtual machine (`Corporate-US-Net`). The asterisk simply means all ports. The rule is named “Danger_Zone” due to the fact that all the ports are open to the VM, which is not ideal in most scenarios.

<img width="2560" height="1440" alt="2nd" src="https://github.com/user-attachments/assets/7530d21a-ff58-40b3-a080-e3893511533f" />
<br />
<br />
<h2></h2>

### VM Firewall

The next step was to log into the virtual machine and turn off all three firewall profiles (Domain, Private, Public), removing any OS-level filtering on top of the NSG already allowing all traffic in.

<img width="2560" height="1440" alt="3rd" src="https://github.com/user-attachments/assets/d0baf45a-60bb-43e9-92f8-2c2fefda9803" />
<br />
<br />
<h2></h2>

### Honeypot/Client Test

The goal of this step was to confirm the virtual machine was reachable over the internet, testing the connection from my local machine. `TcpTestSucceeded` returned `True`, confirming port 3389 was open and reachable – meaning any attacker scanning the internet would be able to reach it too.

<img width="2560" height="1440" alt="4th blanked" src="https://github.com/user-attachments/assets/d572eec1-9f6a-4c1d-957b-ed575da8c8e1" />

### Pre-SIEM Deployment

<b>Log Analytics Workspace (LAW):</b> Following this step, it was crucial to create a Log Analytics Workspace to store and collect data from the virtual machine. <br/>
<b>SIEM:</b> Directly following, the next step was to connect the Log Analytics Workspace to Microsoft Sentinel.
<br />
<br />
<h2></h2>

### Data Connector Setup

The next step was to go to Content Hub in Microsoft Sentinel and install “Windows Security Events”. This gives Sentinel tools like the data connector to collect Windows event logs (authentication attempts) from `Corporate-US-Net`.

<img width="2560" height="1440" alt="5th" src="https://github.com/user-attachments/assets/0f1bd4bb-bae4-43c0-9255-01b6428b8985" />
<br />
<br />
<h2></h2>

### Directly following the last step, the next steps were important for deploying logs:
1. Check “Windows Security Events via AMA” <br/>
2. Click “Open connector page” <br/>
3. Click “Create data collection rule” <br/>
4. Create the rule by selecting the VM (`Corporate-US-Net`)

<img width="2560" height="1440" alt="6th" src="https://github.com/user-attachments/assets/b16f2d7c-1087-4216-971f-484e2a14ac6a" />
<br />
<br />
<h2></h2>

### Confirmation Honeypot/Client

The next part was to proceed to Log Analytics Workspace and wait for attackers to start attempting to get inside the VM (`Corporate-US-Net`). I was able to use the query `SecurityEvent | where EventID == 4625` to pull up an attacker’s information, confirming that the honeypot/client machine was working as intended.

<img width="2560" height="1440" alt="7th" src="https://github.com/user-attachments/assets/fbb467f1-f8c0-4fbf-a505-2dd010b19bfb" />
<br />
<br />
<h2></h2>

### Watchlist Configuration

The next phase was to head over to Microsoft Sentinel and create a Watchlist using Josh Madakor’s GeographicIP file, which allowed geographic filtering in the log repository.

<img width="2560" height="1440" alt="8th" src="https://github.com/user-attachments/assets/021db5c6-97b4-40f7-bd48-1326338aac20" />
<br />
<br />
<h2></h2>

The following picture showed the addition of the Watchlist configuration in my log repository using `_GetWatchlist("GeographicIP")`. With this setup, I was able to see the locations of the attackers.

<img width="2560" height="1440" alt="9th" src="https://github.com/user-attachments/assets/aa742c6b-3e34-4556-b81c-2283a20ee4eb" />
<br />
<br />
<h2></h2>

### Sentinel Workbook Mapping

The next step was creating the “Workbooks”, which was used to geographically map out where the attacks were coming from. From the displayed picture, I was able to get the locations of several attackers from several different areas.

<img width="2560" height="1440" alt="10th" src="https://github.com/user-attachments/assets/e7869c1c-23c1-428b-a845-5eb1e52aba42" />
<br />
<br />
<h2></h2>

### Active Directory & Domain Controller

Subsequently, I provisioned another virtual machine (`DcCorporate`) using Windows Server 2025 to act as a domain controller for the honeypot/client machine, simulating a real-world enterprise environment.

<img width="2560" height="1440" alt="11th" src="https://github.com/user-attachments/assets/16d6a5e2-84fe-402b-b25c-b9d8bc40f69c" />
<br />
<br />
<h2></h2>

**Before:** Directly following the deployment of the DC virtual machine, I installed Active Directory Domain Services (AD DS) and promoted the VM to a domain controller.

<img width="2560" height="1440" alt="12th" src="https://github.com/user-attachments/assets/fcd9214a-9145-46ff-83d9-db6e2b58d17b" />

**After:** I then provisioned a user in an Organizational Unit to later log into "Corporate-US-Net", replicating a real-world user and honeypot environment at the same time. 
<br />
<br />
<h2></h2>

This next part was critical, as it allowed the honeypot/client (`Corporate-US-Net`) to properly resolve and communicate with the domain controller (`DcCorporate`) for Active Directory services.

<img width="2560" height="1440" alt="13th" src="https://github.com/user-attachments/assets/4684d8c4-e914-4d9b-b3a1-374ac5860487" />
<br />
<br />
<h2></h2>

Upon completing the communication phase, I logged into the honeypot/client VM to complete a domain join between the DC and honeypot/client virtual machines.

<img width="2560" height="1440" alt="14th" src="https://github.com/user-attachments/assets/45edcab1-3306-4fe7-8fed-531f46599b47" />
<br />
<br />
<h2></h2>

### Honeypot Account & Access Configuration

To troubleshoot login and log visibility for the new AD account (`dccorp\Clienthoney`) on `Corporate-US-Net`, I added it to the Event Log Readers group, cleared cached Kerberos tickets, and re-enabled the RDP firewall rule.

<img width="2560" height="1440" alt="15th" src="https://github.com/user-attachments/assets/54c57280-fa3e-4748-b0fe-67991b702038" />
<br />
<br />
<h2></h2>

### Attack Simulation VM Deployment

The following phase of the project involved creating another virtual machine (`attack123`) using Ubuntu Server to simulate a real-world attack from Canada East (Quebec).

<img width="2560" height="1440" alt="16th" src="https://github.com/user-attachments/assets/99efae2e-a8e4-40c7-b107-f8b772a3ca65" />
<br />
<br />
<h2></h2>

### Hydra Installation

After deploying the attacking machine, I connected to it via SSH using the created credentials and installed Hydra – a tool that would enable brute-force login attempts, triggering Event ID 4625 on the target machine.

<img width="2560" height="1440" alt="17th" src="https://github.com/user-attachments/assets/5992214c-9f52-4979-9aa9-3fe103ca49a3" />
<br />
<br />
<h2></h2>

### Incident Rules

Before attacking the honeypot/client machine, I created an analytics rule named “Brute Force Attack Detected,” mapped to MITRE ATT&CK’s Credential Access. The rule queried for Event ID 4625, grouped failed logon attempts by account and IP address, and triggered an alert once an account hit 3 or more failures – designed to catch the brute-force attempts I was about to run with Hydra.

<img width="2560" height="1440" alt="18th" src="https://github.com/user-attachments/assets/9242e094-6f15-4eaa-a34b-f8b2fb765c2b" />
<br />
<br />
<h2></h2>

### Confirmation One

This step showed the overall attack map, zoomed to the Canada East region (Quebec, Canada). The 29.5K count from New York served as supporting evidence that attacks against the honeypot/client had already been ongoing for some time, prior to the intentional attack launched from `attack123` using Hydra.

<img width="2560" height="1440" alt="19th" src="https://github.com/user-attachments/assets/78d30dd5-75fb-4378-aeb4-b36f1e35b4a7" />
<br />
<br />
<h2></h2>

### Attack Simulation via Hydra

I launched a brute-force attack against the honeypot/client machine (`Corporate-US-Net`) using Hydra, targeting the `Clienthoney` account with a small password list. The attack ran 4 login attempts, all of which failed (0 valid passwords found), generating 4 Event ID 4625 entries and triggering a single alert from the “Brute Force Attack Detected” rule once the failure threshold was met.

<img width="2560" height="1440" alt="20th" src="https://github.com/user-attachments/assets/66e29fe7-f6ad-485c-9dfe-0d3865ddc349" />
<br />
<br />
<h2></h2>

### Confirmation Two

The attack from `attack123` was successfully mapped on the geographic IP map (Quebec, Canada), confirming that the attack was correctly detected, logged, and geolocated by the SIEM pipeline.

<img width="2560" height="1440" alt="21st" src="https://github.com/user-attachments/assets/92304ed9-c9ef-497a-be73-e2c353e3c03b" />
<br />
<br />
<h2></h2>

### Sentinel Detection

This step confirmed that the end-to-end pipeline worked as intended: the “Brute Force Attack Detected” incident was created, correctly tied to the `Clienthoney` account, and the incident graph visually mapped the connection back to the source IP (20.175.114.187) – the `attack123` VM.

<img width="2560" height="1440" alt="22nd" src="https://github.com/user-attachments/assets/9de80057-5b32-4c49-9c42-d6be90216686" />
<br />
<br />
<h2></h2>

### Incident Response

Directly following the incident alert, I responded by creating a new inbound rule blocking the attacker’s IP address (`20.175.114.187`, the `attack123` VM).

<img width="2560" height="1440" alt="24th" src="https://github.com/user-attachments/assets/64cb7af2-92cd-4eab-b676-a3e3be95b39b" />

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
