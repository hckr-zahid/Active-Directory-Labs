# Lab 05: DNS and DHCP Configuration

**Windows Server 2019 Infrastructure Services and Active Directory Integration**

**Project:** Private Cybersecurity Lab  
**Domain:** `zroots.local`  
**Domain Controller:** `DC01`  
**Lab Status:** Completed

---

## Table of Contents

1. Executive Summary
2. Lab Objectives and Scope
3. Lab Environment and Endpoint Addressing
4. Network Topology
5. DNS Architecture and Reverse Lookup Configuration
6. DNS Resolution and Validation Results
7. DHCP Installation and Active Directory Authorization
8. DHCP Scope Creation and Configuration
9. Kali Linux DHCP Client Test and Lease Verification
10. Kali Linux Restoration and Service Verification
11. Security Lessons and Hardening Recommendations
12. Knowledge Check and Answers
13. Appendix: Important Commands

---

## 1. Executive Summary

This technical report documents the planning, configuration, deployment, and validation of Domain Name System (DNS) and Dynamic Host Configuration Protocol (DHCP) services on Windows Server 2019 within a private Active Directory environment.

The lab uses the `zroots.local` domain, with `DC01` serving as the central domain controller and infrastructure server. DNS provides internal name resolution and Active Directory service discovery, while DHCP enables centrally managed dynamic IPv4 address allocation for compatible lab endpoints.

The lab included configuring an Active Directory-integrated reverse lookup zone, creating and validating Pointer (PTR) records, authorizing the DHCP server in Active Directory, establishing an operational DHCP scope, and testing dynamic address assignment from Kali Linux.

After successful testing, Kali Linux was restored to its original static IP configuration. DNS, DHCP, and Routing and Remote Access Service (RRAS) were also verified as running with automatic startup.

This lab establishes a foundation for subsequent activities involving domain administration, network diagnostics, security testing, and Active Directory hardening.

## 2. Lab Objectives and Scope

The primary objectives of this lab were to:

- Verify the existing DNS forward lookup zones required for Active Directory.
- Create an Active Directory-integrated reverse lookup zone for the `10.1.1.0/24` subnet.
- Configure and validate PTR records for selected lab systems.
- Verify forward DNS resolution and Active Directory LDAP service discovery.
- Install the DHCP Server role on Windows Server 2019.
- Authorize the DHCP server within Active Directory.
- Create and activate a DHCP scope for `10.1.1.100–10.1.1.200`.
- Configure the default gateway, DNS server, DNS suffix, and lease duration.
- Test dynamic IP address assignment using Kali Linux.
- Verify the DHCP lease recorded on `DC01`.
- Restore Kali Linux to its original static network configuration.
- Document the results, troubleshooting considerations, and security lessons.

The scope of this exercise was limited to the private lab environment. Existing static configurations on domain-joined Windows clients were not changed.

## 3. Lab Environment and Endpoint Addressing

The lab operates on VMware with a private host-only network named `MyLabNet`. Windows Server 2019 provides Active Directory Domain Services (AD DS), DNS, DHCP, and RRAS/NAT.

| Endpoint | IPv4 Address | Configuration / Role |
|---|---|---|
| DC01 | `10.1.1.10` | Static; AD DS, DNS, DHCP, RRAS/NAT |
| DC01 WAN | `192.168.0.144` | Bridged interface |
| WIN10-01 | `10.1.1.20` | Static; domain-joined Windows 10 |
| Kali Linux | `10.1.1.30` | Static normally; temporary DHCP test client |
| BlackArch | `10.1.1.40` | Lab security-testing endpoint |
| Windows 11 | `10.1.1.50` | Static; domain-joined Windows 11 |
| Metasploitable | `10.1.1.60` | Vulnerable lab target |

### Domain and network parameters

| Parameter | Value |
|---|---|
| Domain name | `zroots.local` |
| Domain controller | `DC01` |
| Domain distinguished name | `DC=zroots,DC=local` |
| Private subnet | `10.1.1.0/24` |
| Private gateway | `10.1.1.10` |
| Internal DNS server | `10.1.1.10` |
| DHCP scope | `10.1.1.100–10.1.1.200` |
| DHCP lease duration | 8 days |
| DNS suffix | `zroots.local` |

## 4. Network Topology

The lab uses a hybrid network topology that connects the physical network to the Windows Server 2019 virtual machine through a bridged VMware interface. Internal lab endpoints connect through the private host-only network, `MyLabNet`.

**Network path:**

```text
Internet
   |
Physical Router (192.168.0.1)
   |
VMware VMnet0 — Bridged
   |
DC01 — Windows Server 2019
   |-- WAN: 192.168.0.144
   |-- LAN: 10.1.1.10/24
   |-- AD DS, DNS, DHCP, RRAS/NAT
   |
MyLabNet — VMware Host-only Network
10.1.1.0/24
   |
   |-- WIN10-01       10.1.1.20
   |-- Kali Linux     10.1.1.30
   |-- BlackArch      10.1.1.40
   |-- Windows 11     10.1.1.50
   |-- Metasploitable 10.1.1.60
```

VMware's built-in DHCP service is disabled on the private lab network to avoid conflicts with the Windows DHCP Server role.

**Design consideration:** `DC01` acts as the internal gateway and provides routing through RRAS/NAT. Because its WAN interface is bridged to the physical network, firewall rules, routing configuration, and network exposure must be managed carefully.

## 5. DNS Architecture and Reverse Lookup Configuration

DNS is essential to Active Directory because it enables domain clients to locate domain controllers and other directory services. Active Directory uses DNS host records and Service (SRV) records to support domain discovery, authentication, and related operations.

The lab uses the following DNS zones:

- `zroots.local` — forward lookup zone.
- `_msdcs.zroots.local` — Active Directory service discovery zone.
- `1.1.10.in-addr.arpa` — reverse lookup zone for `10.1.1.0/24`.

### 5.1 Creating the reverse lookup zone

An Active Directory-integrated reverse lookup zone was created for the private subnet using PowerShell:

```powershell
Add-DnsServerPrimaryZone `
    -NetworkId "10.1.1.0/24" `
    -ReplicationScope "Domain"
```

This zone allows IP addresses to be mapped back to their corresponding hostnames through PTR records.

### 5.2 Creating PTR records

PTR records were added for the domain controller and the two domain-joined Windows clients:

```powershell
Add-DnsServerResourceRecordPtr `
    -ZoneName "1.1.10.in-addr.arpa" `
    -Name "10" `
    -PtrDomainName "dc01.zroots.local"

Add-DnsServerResourceRecordPtr `
    -ZoneName "1.1.10.in-addr.arpa" `
    -Name "20" `
    -PtrDomainName "WIN10-01.zroots.local"

Add-DnsServerResourceRecordPtr `
    -ZoneName "1.1.10.in-addr.arpa" `
    -Name "50" `
    -PtrDomainName "Windows11.zroots.local"
```

These records establish the following mappings:

| IP address | PTR hostname |
|---|---|
| `10.1.1.10` | `dc01.zroots.local` |
| `10.1.1.20` | `WIN10-01.zroots.local` |
| `10.1.1.50` | `Windows11.zroots.local` |

## 6. DNS Resolution and Validation Results

DNS validation was performed against the internal DNS server at `10.1.1.10`.

### 6.1 Reverse DNS validation

The following commands were used to verify reverse lookups:

```powershell
Resolve-DnsName 10.1.1.10 -Server 10.1.1.10
Resolve-DnsName 10.1.1.20 -Server 10.1.1.10
Resolve-DnsName 10.1.1.50 -Server 10.1.1.10
```

**Observed results:**

| Test | Expected result | Recorded result |
|---|---|---|
| `10.1.1.10` | `dc01.zroots.local` | Successful |
| `10.1.1.20` | `WIN10-01.zroots.local` | Successful |
| `10.1.1.50` | `Windows11.zroots.local` | Successful |

### 6.2 Forward DNS validation

The domain controller's host record was checked using:

```powershell
Resolve-DnsName dc01.zroots.local -Server 10.1.1.10
```

The recorded result returned the internal address `10.1.1.10`.

### 6.3 Active Directory service discovery

The LDAP SRV record was queried to confirm that the domain controller could be located through DNS:

```powershell
Resolve-DnsName `
    _ldap._tcp.dc._msdcs.zroots.local `
    -Type SRV `
    -Server 10.1.1.10
```

The query identified `dc01.zroots.local` on LDAP port `389`.

### 6.4 External DNS resolution

External name resolution was tested through the internal DNS server:

```powershell
Resolve-DnsName www.microsoft.com -Server 10.1.1.10
```

The query succeeded. External resolution used DNS root hints; no explicit DNS forwarders were configured for this test.

**Result:** The recorded DNS tests successfully validated the selected reverse records, the domain controller's internal host record, Active Directory LDAP service discovery, and external hostname resolution.

## 7. DHCP Installation and Active Directory Authorization

The DHCP Server role was installed on Windows Server 2019 using PowerShell:

```powershell
Install-WindowsFeature DHCP -IncludeManagementTools
```

**Recorded installation result:**

- Success: `True`
- Restart required: `No`
- Exit code: `Success`

### 7.1 Authorizing the DHCP server

The DHCP server was authorized in Active Directory using its internal DNS name and private LAN address:

```powershell
Add-DhcpServerInDC `
    -DnsName "DC01.zroots.local" `
    -IPAddress 10.1.1.10
```

Authorization helps prevent unauthorized Windows DHCP servers from distributing leases in an Active Directory-managed environment.

The authorization was verified with:

```powershell
Get-DhcpServerInDC
```

The recorded output identified `DC01.zroots.local` at `10.1.1.10`.

## 8. DHCP Scope Creation and Configuration

A DHCP scope named `MyLabNet-DHCP` was created for the private subnet. Its address pool was deliberately separated from the static addresses used by the core lab infrastructure.

### 8.1 Creating the DHCP scope

```powershell
Add-DhcpServerv4Scope `
    -Name "MyLabNet-DHCP" `
    -StartRange 10.1.1.100 `
    -EndRange 10.1.1.200 `
    -SubnetMask 255.255.255.0 `
    -State Active
```

The scope was activated successfully.

### 8.2 Configuring DHCP options

The DNS server and DNS suffix were configured as follows:

```powershell
Set-DhcpServerv4OptionValue `
    -ScopeId 10.1.1.0 `
    -DnsServer 10.1.1.10 `
    -DnsDomain "zroots.local"
```

The default gateway was configured separately:

```powershell
Set-DhcpServerv4OptionValue `
    -ScopeId 10.1.1.0 `
    -Router 10.1.1.10
```

### 8.3 DHCP configuration summary

| Parameter | Configured value |
|---|---|
| Scope name | `MyLabNet-DHCP` |
| Scope network | `10.1.1.0/24` |
| Start address | `10.1.1.100` |
| End address | `10.1.1.200` |
| Total available addresses | 101 |
| Scope state | Active |
| Option 003 — Router | `10.1.1.10` |
| Option 006 — DNS Server | `10.1.1.10` |
| Option 015 — DNS Domain | `zroots.local` |
| Option 051 — Lease Duration | 691,200 seconds (8 days) |

The scope configuration and options were inspected using:

```powershell
Get-DhcpServerv4Scope

Get-DhcpServerv4OptionValue -ScopeId 10.1.1.0

Get-DhcpServerv4ScopeStatistics -ScopeId 10.1.1.0
```

Following the Kali Linux DHCP test, the recorded scope statistics showed 100 free addresses and one address in use.

## 9. Kali Linux DHCP Client Test and Lease Verification

Kali Linux normally uses the static address `10.1.1.30/24`. For this test, its NetworkManager connection, `Wired connection 1`, was temporarily switched to automatic IPv4 configuration.

### 9.1 Enabling DHCP on Kali Linux

The following commands were executed on Kali Linux:

```bash
sudo nmcli connection modify "Wired connection 1" ipv4.method auto
sudo nmcli connection up "Wired connection 1"
```

The assigned network configuration was then inspected:

```bash
ip -4 address show dev eth0
ip route
nmcli device show eth0
```

### 9.2 Observed DHCP configuration

Kali Linux successfully obtained the following configuration from the Windows DHCP server:

| Parameter | Observed value |
|---|---|
| Assigned IPv4 address | `10.1.1.100/24` |
| Default gateway | `10.1.1.10` |
| DNS server | `10.1.1.10` |
| DNS suffix | `zroots.local` |

### 9.3 Verifying the lease on DC01

The DHCP lease database was inspected on the Windows Server:

```powershell
Get-DhcpServerv4Lease -ScopeId 10.1.1.0 |
    Format-Table IPAddress, HostName, ClientId, AddressState, LeaseExpiryTime -AutoSize
```

The recorded lease was:

| Field | Recorded value |
|---|---|
| IP address | `10.1.1.100` |
| Hostname | `kali.zroots.local` |
| Client ID / MAC | `00-0c-29-ed-59-59` |
| Address state | Active |
| Lease expiry | `10/24/2026 12:45:54 AM` |

The lease confirmed that Kali had received an address from the configured dynamic pool and that the DHCP server had recorded the assignment.

### 9.4 Connectivity and DNS tests

The following tests were performed on Kali Linux:

```bash
ping -c 4 10.1.1.10
ping -c 4 1.1.1.1
nslookup dc01.zroots.local 10.1.1.10
nslookup www.microsoft.com 10.1.1.10
```

**Recorded results:**

- Connectivity to `10.1.1.10`: 4 packets received, 0% packet loss.
- Connectivity to `1.1.1.1`: 4 packets received, 0% packet loss.
- Internal DNS lookup for `dc01.zroots.local`: successful.
- External DNS lookup for `www.microsoft.com`: successful.

These results confirmed the tested client-to-server connectivity, external IP reachability, and DNS resolution at the time of testing. They do not independently establish the security or correctness of every RRAS, NAT, or firewall rule.

## 10. Kali Linux Restoration and Service Verification

After completing the DHCP test, Kali Linux was restored to its original static configuration to preserve the established lab addressing scheme.

### 10.1 Restoring the static configuration

The following commands were executed on Kali Linux:

```bash
sudo nmcli connection modify "Wired connection 1" \
    ipv4.method manual \
    ipv4.addresses 10.1.1.30/24 \
    ipv4.gateway 10.1.1.10 \
    ipv4.dns 10.1.1.10 \
    ipv4.dns-search ""

sudo nmcli connection up "Wired connection 1"
```

The final configuration was checked using:

```bash
ip -4 address show dev eth0
ip route
nmcli device show eth0
```

**Recorded final state:**

- IPv4 address: `10.1.1.30/24`
- Default gateway: `10.1.1.10`
- DNS server: `10.1.1.10`
- NetworkManager connection: `Wired connection 1`

### 10.2 DHCP lease note

After Kali was restored to its static address, DC01 continued to display the earlier `10.1.1.100` lease as active. The lease record was not deleted.

A server-side lease record alone does not prove that the client is still using the leased address. The final Kali interface and routing checks confirmed that the client had returned to its static configuration.

### 10.3 Verifying core services

The following command was used on DC01:

```powershell
Get-Service DHCPServer, DNS, RemoteAccess |
    Format-Table Name, Status, StartType -AutoSize
```

The recorded results showed all three services running with automatic startup:

| Service | Status | Startup type |
|---|---|---|
| DHCP Server (`DHCPServer`) | Running | Automatic |
| DNS Server (`DNS`) | Running | Automatic |
| Routing and Remote Access (`RemoteAccess`) | Running | Automatic |

**Result:** The DHCP test was completed, Kali's original static configuration was restored, and the core infrastructure services remained operational.

## 11. Security Lessons and Hardening Recommendations

### 11.1 Active Directory DNS integration

Domain-joined systems should use the designated internal Active Directory DNS server for domain resolution and service discovery. Incorrect DNS settings can interfere with domain controller discovery, authentication, and Group Policy processing.

### 11.2 DHCP authorization

Authorizing the Windows DHCP server in Active Directory helps reduce the risk of unauthorized Windows DHCP servers distributing network configuration to domain clients. Authorization should be combined with appropriate network controls and monitoring.

### 11.3 Address-pool segmentation

Keeping static infrastructure addresses outside the dynamic DHCP pool reduces the likelihood of address conflicts. Document static assignments, DHCP ranges, and any exclusions or reservations.

### 11.4 Network isolation and gateway security

The private host-only network limits direct exposure of lab endpoints. However, DC01's bridged WAN interface connects it to the physical network. Apply least-privilege firewall rules, restrict administrative access, and review RRAS/NAT configuration regularly.

### 11.5 DNS record accuracy

Maintain accurate forward and reverse DNS records. Review records when systems are renamed, removed, or assigned new addresses. Do not assume that a successful query alone verifies every record in a zone.

### 11.6 Evidence-based validation

Record the actual output of validation commands and preserve screenshots of relevant configuration. Distinguish between successful individual tests and a comprehensive security assessment.

### 11.7 Preserve established lab configurations

Use temporary client changes for testing and restore the original configuration afterward. Avoid changing static addresses on existing domain-joined systems unnecessarily.

## 12. Knowledge Check and Answers

**Q1. What is the purpose of DNS in an Active Directory environment?**

DNS resolves hostnames to IP addresses and enables clients to locate domain controllers and directory services through records such as LDAP SRV records.

**Q2. What is the purpose of a reverse lookup zone and PTR record?**

A reverse lookup zone supports queries that map an IP address to a hostname. A PTR record stores the hostname associated with an IP address.

**Q3. What does DHCP Option 003 provide?**

Option 003 supplies the client's default gateway. In this lab, the configured gateway is `10.1.1.10`.

**Q4. What do DHCP Options 006 and 015 provide?**

Option 006 supplies the DNS server address (`10.1.1.10`), while Option 015 supplies the DNS domain suffix (`zroots.local`).

**Q5. Why is DHCP authorization important in Active Directory?**

It helps prevent unauthorized Windows DHCP servers from distributing leases in an Active Directory-managed environment.

**Q6. Why was Kali Linux restored to `10.1.1.30` after testing?**

The static address is part of the established lab addressing plan. Restoring it preserves consistent connectivity for later security exercises.

**Q7. Does a successful ping to an external IP address prove that DNS is working?**

No. A ping to an IP address tests IP connectivity, not hostname resolution. DNS must be tested separately using tools such as `nslookup` or `Resolve-DnsName`.

**Q8. What security consideration arises from using a bridged WAN interface on DC01?**

The server is connected to the physical network, increasing the importance of firewall policy, restricted administration, and careful routing/NAT configuration.

## 13. Appendix: Important Commands

### 13.1 DNS configuration and validation

Create the reverse lookup zone:

```powershell
Add-DnsServerPrimaryZone `
    -NetworkId "10.1.1.0/24" `
    -ReplicationScope "Domain"
```

Create a PTR record for DC01:

```powershell
Add-DnsServerResourceRecordPtr `
    -ZoneName "1.1.10.in-addr.arpa" `
    -Name "10" `
    -PtrDomainName "dc01.zroots.local"
```

Validate reverse DNS:

```powershell
Resolve-DnsName 10.1.1.10 -Server 10.1.1.10
```

Validate the domain controller's host record:

```powershell
Resolve-DnsName dc01.zroots.local -Server 10.1.1.10
```

Validate the LDAP SRV record:

```powershell
Resolve-DnsName `
    _ldap._tcp.dc._msdcs.zroots.local `
    -Type SRV `
    -Server 10.1.1.10
```

### 13.2 DHCP deployment and verification

Install the DHCP Server role:

```powershell
Install-WindowsFeature DHCP -IncludeManagementTools
```

Authorize the server:

```powershell
Add-DhcpServerInDC `
    -DnsName "DC01.zroots.local" `
    -IPAddress 10.1.1.10
```

Create the scope:

```powershell
Add-DhcpServerv4Scope `
    -Name "MyLabNet-DHCP" `
    -StartRange 10.1.1.100 `
    -EndRange 10.1.1.200 `
    -SubnetMask 255.255.255.0 `
    -State Active
```

Configure DNS settings:

```powershell
Set-DhcpServerv4OptionValue `
    -ScopeId 10.1.1.0 `
    -DnsServer 10.1.1.10 `
    -DnsDomain "zroots.local"
```

Configure the default gateway:

```powershell
Set-DhcpServerv4OptionValue `
    -ScopeId 10.1.1.0 `
    -Router 10.1.1.10
```

Inspect DHCP leases:

```powershell
Get-DhcpServerv4Lease -ScopeId 10.1.1.0
```

Inspect scope statistics:

```powershell
Get-DhcpServerv4ScopeStatistics -ScopeId 10.1.1.0
```

Verify service status:

```powershell
Get-Service DHCPServer, DNS, RemoteAccess
```

### 13.3 Kali Linux network testing

Switch to DHCP:

```bash
sudo nmcli connection modify "Wired connection 1" ipv4.method auto
sudo nmcli connection up "Wired connection 1"
```

Test connectivity and DNS:

```bash
ping -c 4 10.1.1.10
ping -c 4 1.1.1.1
nslookup dc01.zroots.local 10.1.1.10
nslookup www.microsoft.com 10.1.1.10
```

Restore the static configuration using the commands documented in Section 10.1.

---

## Lab Completion Summary

| Lab component | Result |
|---|---|
| Reverse lookup zone | Created and verified |
| PTR records | Successfully validated |
| DC01 forward lookup | Successful |
| Active Directory LDAP SRV lookup | Successful |
| External DNS resolution | Successful |
| DHCP role installation | Successful |
| Active Directory DHCP authorization | Verified |
| DHCP scope and options | Configured and verified |
| Kali dynamic lease | Successful |
| Internal and external-IP connectivity tests | Successful |
| Kali static configuration restoration | Verified |
| DNS, DHCP, and RRAS services | Running; Automatic startup |

**Overall status: Lab 05 completed.**

**Next lab:** Lab 06 — File Server and Permissions.

The next lab will cover creating a dedicated test share, configuring share and NTFS permissions, validating access through domain users and groups, and applying the principle of least privilege.

---

*Private Cybersecurity Lab Handbook — Lab 05 Report*

*Documentation note: This report reflects the recorded configuration and test results for this lab. Repeat the verification commands when performing a current-state audit. Redact sensitive information before sharing screenshots or publishing the report in a public repository.*
