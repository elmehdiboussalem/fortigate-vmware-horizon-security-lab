# VMware Horizon 8 deployment notes

Environment: ESXi 7 physical host connected to the server-room switch, itself uplinked to the federating switch. Virtual servers on ESXi, each dedicated to a single role (AD/DNS, Connection Server, Agent), plus a Windows 10 client VM for testing.

## 1. Active Directory / DNS
- Windows Server 2019, static IP
- Roles: AD DS + DNS Server, promoted to domain controller
- Accounts: one admin account for the Connection Server installation, one for the Agent, standard users in *Student* and *Prof* groups

## 2. Connection Server
- Join the domain with the dedicated account
- Run the Horizon 8 Connection Server installer
- Options: **Horizon Standard Server**, **Install HTML Access**
- Set the initial admin password, let the installer configure the Windows firewall, confirm the domain
- Access the console at `https://<connection-server>/admin`

## 3. Agent server (RDS mode)
- Join the domain with the Agent account
- Run the Horizon Agent installer in **RDS Mode** (publishes applications / shared sessions, unlike Desktop mode)
- Reboot, re-run the installer to finish configuration, accept licence, choose IPv4
- Keep default components, enable **USB Redirection**
- *Register with Horizon Connection Server*: Connection Server address + domain credentials
- Restart

## 4. RDS farm and published applications
1. Install the business applications on the Agent server (Office suite, etc.)
2. In Horizon Console: *Inventory → Machines → RDS Hosts* → confirm the Agent is registered
3. *Inventory → Farms → Add* → **Manual Farm**
4. Publish applications from the farm

## 5. Validation
- Connect with the Horizon client from PC and tablet
- Check AD policy enforcement (lockout after failed attempts)
- Measured latency < 30 ms

## Troubleshooting log
| Symptom | Cause / action |
|---|---|
| USB redirection latency | Adjust Horizon Agent settings to prioritize USB traffic |
| VMs fail to join the desktop pool | Verify DHCP and DNS settings and AD permissions |
