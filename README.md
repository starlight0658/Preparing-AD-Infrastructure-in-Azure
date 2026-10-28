# Preparing Active Directory Infrastructure in Azure

**Hands-on Microsoft Azure / Windows infrastructure lab** · Azure Virtual Machines · Windows Server · Windows 11 · Networking · DNS · PowerShell

## Overview

Built the Azure infrastructure for a future Windows Active Directory environment using a Windows Server VM (`DC-1`) and a Windows client VM (`Client-1`). Configured a shared virtual network, static server IP addressing, client DNS settings, and remote desktop access. Troubleshot an ICMP connectivity failure and verified end-to-end reachability.

**Skills demonstrated:** Azure resource groups · Virtual networks and subnets · Windows Server and Windows 11 VMs · NIC/IP management · Static IPv4 addressing · DNS client configuration · Remote Desktop (RDP) · Windows Defender Firewall · PowerShell networking diagnostics · Network troubleshooting

## Environment

| Component | Configuration shown in the lab |
| --- | --- |
| Cloud platform | Microsoft Azure — East US 2 |
| Resource group | `Active-Directory-Lab` |
| Virtual network | `Active-Directory-VNet` — `10.0.0.0/16` |
| Subnet | `default` — `10.0.0.0/24` |
| Windows Server VM | `DC-1` — Windows Server 2025 Datacenter — `10.0.0.4` (**static**) |
| Windows client VM | `Client-1` — Windows 11 Pro — `10.0.0.5` (dynamic in screenshots) |
| Client DNS server setting | `10.0.0.4` (DC-1 private address) |
| VM size | Standard D2ads v6 (2 vCPU, 8 GiB RAM) |
| Remote management | Windows App for macOS / RDP |

```text
                Azure — Active-Directory-Lab (East US 2)
                  Active-Directory-VNet 10.0.0.0/16
                          default 10.0.0.0/24
                          /                \
        DC-1 (Windows Server)          Client-1 (Windows 11)
        10.0.0.4 — static               10.0.0.5 — dynamic
                                         DNS → 10.0.0.4
                          ^                 |
                          +---- ICMP -------+
        RDP administration from Windows App for macOS
```

## Validated results

- **Shared network:** Both machines were placed on `Active-Directory-VNet` / `default` in East US 2.
- **Stable server address:** DC-1's NIC changed from dynamically assigned to **static `10.0.0.4`**.
- **Client DNS configured:** Client-1 was configured to use **`10.0.0.4`**; `ipconfig /all` displayed that address in **DNS Servers**.
- **Connectivity restored:** Initial `ping 10.0.0.4` attempts timed out; later testing returned **4 replies from 4 packets (0% packet loss)** with approximately 1 ms response time after the lab firewall changes.

**Connectivity confirmation — Client-1 → DC-1**

![Four successful replies to the DC-1 private IP from Client-1](screenshots/42-client-ping-success.png)

**DNS verification — Client-1**

![Client-1 ipconfig output identifying 10.0.0.4 as DNS server](screenshots/43-client-ipconfig-dns.png)

## Configuration and troubleshooting

1. Created `Active-Directory-Lab` and the `Active-Directory-VNet` network with the `default` subnet.
2. Deployed Windows Server `DC-1` and Windows client `Client-1` in the same Azure virtual network.
3. Set DC-1's private NIC address to static `10.0.0.4`, preserving a consistent target for client configuration.
4. Used Windows App for macOS to connect to the remote VMs and inspect Windows networking and firewall settings.
5. Configured Client-1's custom NIC DNS server to `10.0.0.4`, saved it, and restarted the client to apply settings.
6. Used PowerShell `ping 10.0.0.4` to test reachability. Initial tests failed; after changing the Windows Server firewall for the lab, the same test succeeded.
7. Ran `ipconfig /all` on Client-1 and verified that **DNS Servers** listed `10.0.0.4`.

```powershell
ping 10.0.0.4
ipconfig /all
```

### Scope and security notes

- This lab **prepares infrastructure for Active Directory**. These screenshots do **not** establish that AD DS was installed, a forest/domain was created, or Client-1 joined a domain. Those steps are outside this lab's evidence.
- Windows Defender Firewall was temporarily **disabled for connectivity testing**. For a production environment, keep the firewall enabled and use scoped inbound ICMP rules rather than disabling profiles.
- Inbound RDP (TCP 3389) was opened for the test environment; real deployments should restrict source IPs and preferably use a secure access method such as Azure Bastion or just-in-time access.
- Azure VMs were retained for subsequent labs rather than deleted in this exercise. Stop/deallocate unused VMs to limit Azure charges.

## Screenshot evidence

**43 original screenshots**, preserved and renamed in chronological capture order. Expand any section to review the Azure Portal, Windows Server, and Client-1 configuration evidence. The initial settings screens and later confirmed results are distinguished below.

<details>
<summary><strong>Azure resource group and virtual network</strong> — images 01–05</summary>

Prepared a dedicated Azure resource group, virtual network, and subnet for both systems.

**01. Resource group settings reviewed for Active-Directory-Lab in East US 2**

![Resource group settings reviewed for Active-Directory-Lab in East US 2](screenshots/01-resource-group-review.png)

**02. Active-Directory-Lab resource group visible in Azure**

![Active-Directory-Lab resource group visible in Azure](screenshots/02-resource-group-created.png)

**03. Configured Active-Directory-VNet in East US 2**

![Configured Active-Directory-VNet in East US 2](screenshots/03-virtual-network-basics.png)

**04. Reviewed VNet address space 10.0.0.0/16 and default subnet 10.0.0.0/24**

![Reviewed VNet address space 10.0.0.0/16 and default subnet 10.0.0.0/24](screenshots/04-virtual-network-review.png)

**05. Verified the virtual network in Azure**

![Verified the virtual network in Azure](screenshots/05-virtual-network-overview.png)

</details>

<details>
<summary><strong>Windows Server and Windows client provisioning</strong> — images 06–13</summary>

Configured DC-1 and Client-1 in the same Azure region and virtual network. The actual VM images shown are Windows Server 2025 Datacenter and Windows 11 Pro.

**06. Prepared DC-1 with Windows Server 2025 Datacenter**

![Prepared DC-1 with Windows Server 2025 Datacenter](screenshots/06-server-vm-basics.png)

**07. Configured server administrator and test RDP access**

![Configured server administrator and test RDP access](screenshots/07-server-vm-admin-rdp.png)

**08. Attached DC-1 to Active-Directory-VNet and default subnet**

![Attached DC-1 to Active-Directory-VNet and default subnet](screenshots/08-server-vm-networking.png)

**09. Reviewed Windows Server VM deployment configuration**

![Reviewed Windows Server VM deployment configuration](screenshots/09-server-vm-review.png)

**10. Prepared Client-1 with Windows 11 Pro**

![Prepared Client-1 with Windows 11 Pro](screenshots/10-client-vm-basics.png)

**11. Configured client administrator and test RDP access**

![Configured client administrator and test RDP access](screenshots/11-client-vm-admin-rdp.png)

**12. Attached Client-1 to the same virtual network and subnet**

![Attached Client-1 to the same virtual network and subnet](screenshots/12-client-vm-networking.png)

**13. Reviewed Windows client VM deployment configuration**

![Reviewed Windows client VM deployment configuration](screenshots/13-client-vm-review.png)

</details>

<details>
<summary><strong>Server networking and static private IPv4</strong> — images 14–19</summary>

Verified the server network configuration, then changed DC-1 from dynamic to static private IPv4 allocation at `10.0.0.4`.

**14. Verified DC-1 Windows Server VM and private IP 10.0.0.4**

![Verified DC-1 Windows Server VM and private IP 10.0.0.4](screenshots/14-server-vm-running.png)

**15. Reviewed DC-1 network security group and inbound RDP rule**

![Reviewed DC-1 network security group and inbound RDP rule](screenshots/15-server-network-security.png)

**16. Checked the initial dynamic private IPv4 assignment on DC-1 NIC**

![Checked the initial dynamic private IPv4 assignment on DC-1 NIC](screenshots/16-server-nic-dynamic-ip.png)

**17. Selected static allocation for DC-1 private IP 10.0.0.4**

![Selected static allocation for DC-1 private IP 10.0.0.4](screenshots/17-server-nic-static-settings.png)

**18. Applied the private IP configuration change to DC-1 NIC**

![Applied the private IP configuration change to DC-1 NIC](screenshots/18-server-nic-update.png)

**19. Confirmed DC-1 private IP 10.0.0.4 is static**

![Confirmed DC-1 private IP 10.0.0.4 is static](screenshots/19-server-nic-static-confirmed.png)

</details>

<details>
<summary><strong>Remote server access and firewall testing</strong> — images 20–27</summary>

Connected to DC-1 from macOS through Windows App/RDP, opened Server Manager, and inspected Windows Defender Firewall profiles for lab connectivity troubleshooting.

**20. Added DC-1 RDP connection in Windows App on macOS**

![Added DC-1 RDP connection in Windows App on macOS](screenshots/20-server-rdp-setup.png)

**21. Connected to DC-1 Windows Server and opened Server Manager**

![Connected to DC-1 Windows Server and opened Server Manager](screenshots/21-server-desktop.png)

**22. Opened Windows search to launch the Run dialog**

![Opened Windows search to launch the Run dialog](screenshots/22-server-search-run.png)

**23. Launched Windows Defender Firewall console using wf.msc**

![Launched Windows Defender Firewall console using wf.msc](screenshots/23-server-run-firewall-console.png)

**24. Reviewed Windows Defender Firewall with Advanced Security**

![Reviewed Windows Defender Firewall with Advanced Security](screenshots/24-server-firewall-overview.png)

**25. Reviewed firewall Domain profile settings during connectivity testing**

![Reviewed firewall Domain profile settings during connectivity testing](screenshots/25-server-firewall-domain-settings.png)

**26. Reviewed firewall Private profile settings during connectivity testing**

![Reviewed firewall Private profile settings during connectivity testing](screenshots/26-server-firewall-private-settings.png)

**27. Reviewed firewall Public profile settings during connectivity testing**

![Reviewed firewall Public profile settings during connectivity testing](screenshots/27-server-firewall-public-settings.png)

</details>

<details>
<summary><strong>Client networking and custom DNS</strong> — images 28–35</summary>

Verified both VMs, configured Client-1 to use the server private IP `10.0.0.4` as its custom DNS resolver, saved settings, and initiated a client restart.

**28. Reviewed Azure VM inventory with both DC-1 and Client-1**

![Reviewed Azure VM inventory with both DC-1 and Client-1](screenshots/28-vm-inventory.png)

**29. Rechecked DC-1 private IP 10.0.0.4 and Azure networking**

![Rechecked DC-1 private IP 10.0.0.4 and Azure networking](screenshots/29-server-vm-details.png)

**30. Reviewed Client-1 network security group and private IP 10.0.0.5**

![Reviewed Client-1 network security group and private IP 10.0.0.5](screenshots/30-client-network-security.png)

**31. Confirmed Client-1 dynamic private IP 10.0.0.5**

![Confirmed Client-1 dynamic private IP 10.0.0.5](screenshots/31-client-nic-ip-details.png)

**32. Set Client-1 NIC custom DNS server to 10.0.0.4**

![Set Client-1 NIC custom DNS server to 10.0.0.4](screenshots/32-client-dns-edit.png)

**33. Verified Azure saved Client-1 custom DNS setting**

![Verified Azure saved Client-1 custom DNS setting](screenshots/33-client-dns-save-confirmed.png)

**34. Initiated restart of Client-1 to apply DNS settings**

![Initiated restart of Client-1 to apply DNS settings](screenshots/34-client-restart.png)

**35. Verified Client-1 was running in Azure**

![Verified Client-1 was running in Azure](screenshots/35-client-vm-running.png)

</details>

<details>
<summary><strong>Client access, connectivity troubleshooting, and validation</strong> — images 36–43</summary>

Connected to Client-1, observed initial ICMP timeouts, adjusted the server firewall for this controlled test, and verified successful ICMP replies and the client DNS entry.

**36. Added Client-1 RDP connection in Windows App on macOS**

![Added Client-1 RDP connection in Windows App on macOS](screenshots/36-client-rdp-setup.png)

**37. Connected to the Client-1 Windows 11 desktop**

![Connected to the Client-1 Windows 11 desktop](screenshots/37-client-desktop.png)

**38. Rechecked DC-1 virtual machine connectivity details**

![Rechecked DC-1 virtual machine connectivity details](screenshots/38-server-network-rechecked.png)

**39. Observed initial ping timeouts reaching DC-1 at 10.0.0.4**

![Observed initial ping timeouts reaching DC-1 at 10.0.0.4](screenshots/39-client-ping-timeouts.png)

**40. Reviewed DC-1 firewall profile settings during ping troubleshooting**

![Reviewed DC-1 firewall profile settings during ping troubleshooting](screenshots/40-server-firewall-profile-test.png)

**41. Verified Windows Defender Firewall was off for all three profiles in this lab-only test**

![Verified Windows Defender Firewall was off for all three profiles in this lab-only test](screenshots/41-server-firewall-off.png)

**42. Verified ping to 10.0.0.4 succeeded: four replies and zero packet loss**

![Verified ping to 10.0.0.4 succeeded: four replies and zero packet loss](screenshots/42-client-ping-success.png)

**43. Verified ipconfig /all on Client-1 lists DNS server 10.0.0.4**

![Verified ipconfig /all on Client-1 lists DNS server 10.0.0.4](screenshots/43-client-ipconfig-dns.png)

</details>

---

*Portfolio project: Azure virtual networking, Windows infrastructure, and hands-on troubleshooting. All screenshots were captured in the lab environment.*
