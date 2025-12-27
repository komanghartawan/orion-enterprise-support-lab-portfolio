# LAB-01 Core Identity - Runbook

## Purpose
Build a minimal, deterministic Active Directory lab with one Domain Controller (AD DS, DNS, DHCP) and two Windows 11 domain-joined clients on an isolated VMware host-only network.

## Baseline
- Environment posture: isolated lab, no NAT, no Internet gateway, no default gateway on VMs.
- VMware network: Host-only LAN-LAB (VMnet10)
- Subnet: 10.70.10.0/24
- VMware DHCP: Disabled
- Domain: orion.lab (NetBIOS: ORION)

### Systems
- ORION-L-DC01 (Windows Server 2025)
  - IPv4: 10.70.10.10/24 (static)
  - Default gateway: none
  - DNS: 10.70.10.10
  - Roles: AD DS, DNS, DHCP
- ORION-L-W11-01 (Windows 11 Pro)
  - DHCP reservation: 10.70.10.100
- ORION-L-W11-02 (Windows 11 Pro)
  - DHCP reservation: 10.70.10.101

### DHCP (on DC01)
- Scope: 10.70.10.100-10.70.10.199
- Options (scope):
  - 003 Router: not configured
  - 006 DNS Servers: 10.70.10.10
  - 015 DNS Domain Name: orion.lab
- DNS update behavior baseline:
  - DynamicUpdates: OnClientRequest
  - UpdateDnsRRForOlderClients: True
  - DeleteDnsRROnLeaseExpiry: True
  - DisableDnsPtrRRUpdate: False
  - NameProtection: False

## Evidence and documentation rules
- Evidence folder: `./evidence/`
- Naming: `lab-01_nn_system_what.ext` (lowercase only, underscores only)
- TXT evidence must be UTF-8, captured as raw command output.
- Capture locally on each machine to:
  - `C:\orion-evidence\LAB-01`
  then copy into `./evidence/`.
- Commands used to generate evidence are documented in:
  - `./commands/commands.md`

---

## 0) Prerequisites and rollback

### Prerequisites
- VMware Workstation installed on host.
- Windows Server 2025 ISO for DC01.
- Windows 11 Pro ISO for W11-01 and W11-02.

### Rollback checkpoints (VM snapshots)
Take snapshots at these points:
1) After OS install and basic updates (each VM)
   - Snapshot name: `post-os-install`
2) Before DC promotion (DC01)
   - Snapshot name: `pre-ad-promotion`
3) Immediately after DC promotion succeeds (DC01)
   - Snapshot name: `post-ad-promotion`
4) Before domain join (W11-01, W11-02)
   - Snapshot name: `pre-domain-join`

Rollback method:
- Revert to the most recent snapshot taken before the failing step.

---

## 1) Host network configuration (VMware)

### Configure LAN-LAB (VMnet10) as host-only with VMware DHCP disabled
1) Open VMware Workstation.
2) Edit - Virtual Network Editor.
3) Select VMnet10 and set it as Host-only.
4) Set:
   - Subnet IP: 10.70.10.0
   - Subnet mask: 255.255.255.0
5) Ensure:
   - Connect a host virtual adapter to this network: enabled
   - Use local DHCP service to distribute IP address to VMs: disabled
6) Apply changes.

Evidence:
- `./evidence/lab-01_01_host_vmnet10_lan_lab.png`

---

## 2) VM creation and network attachment

### Network adapter (all VMs: DC01, W11-01, W11-02)
1) VM - Settings.
2) Network Adapter:
   - Custom: Specific virtual network
   - VMnet10 (LAN-LAB)
3) Ensure:
   - Connected: enabled
   - Connect at power on: enabled

---

## 3) Build DC01 (ORION-L-DC01)

### 3.1 Install OS and set hostname
1) Install Windows Server on DC01.
2) Rename computer to `ORION-L-DC01`.
3) Reboot.

### 3.2 Configure static IPv4 (no default gateway)
1) Open NIC properties.
2) IPv4 settings:
   - IP address: 10.70.10.10
   - Subnet mask: 255.255.255.0
   - Default gateway: leave blank
   - Preferred DNS server: 10.70.10.10
3) Apply changes.

Verification:
- `ipconfig /all` shows IPv4 10.70.10.10, DNS 10.70.10.10, gateway blank.

Evidence:
- `./evidence/lab-01_02_dc01_ipconfig_all.png`
- `./evidence/lab-01_13_dc01_ipconfig_all.txt`

### 3.3 Install AD DS, DNS, DHCP roles
1) Server Manager - Manage - Add Roles and Features.
2) Installation type:
   - Role-based or feature-based installation
3) Select server:
   - ORION-L-DC01
4) Add roles:
   - Active Directory Domain Services
   - DNS Server
   - DHCP Server
5) Accept required features when prompted.
6) Install.

---

## 4) Promote DC01 (new forest) and validate core AD

### 4.1 Promote to Domain Controller
1) Server Manager - Notifications - Promote this server to a domain controller.
2) Deployment:
   - Add a new forest
   - Root domain name: orion.lab
3) Domain Controller options:
   - DNS: enabled
   - Global Catalog: enabled
   - RODC: disabled
   - Set DSRM password
4) NetBIOS name:
   - ORION (confirm)
5) Paths:
   - Keep defaults (NTDS, logs, SYSVOL)
6) Install and reboot.

Verification (PowerShell on DC01):
- `Get-ADDomain` and `Get-ADForest` complete successfully.

Evidence:
- `./evidence/lab-01_20_dc01_get_addomain.txt`
- `./evidence/lab-01_21_dc01_get_adforest.txt`
- `./evidence/lab-01_29_dc01_get_addomaincontroller.txt`
- `./evidence/lab-01_39_dc01_operation_master_roles.txt`
- `./evidence/lab-01_40_dc01_ad_partitions.txt`

### 4.2 DNS forward zones and DC record
1) DNS Manager.
2) Confirm Forward Lookup Zones contain:
   - orion.lab
   - _msdcs.orion.lab
3) Confirm A record exists:
   - ORION-L-DC01 -> 10.70.10.10

Evidence:
- `./evidence/lab-01_05_dc01_dns_forward_zone.png`

---

## 5) Configure reverse DNS zone (DC01)

1) DNS Manager - Reverse Lookup Zones - New Zone.
2) Zone type:
   - Primary zone
   - Store the zone in Active Directory: enabled
3) Replication:
   - To all DNS servers in this domain
4) Network ID:
   - 10.70.10
5) Dynamic updates:
   - Secure only
6) Finish.

Evidence:
- `./evidence/lab-01_06_dc01_dns_reverse_zone.png`
- `./evidence/lab-01_07_dc01_dns_reverse_zone_properties.png`

---

## 6) Configure DHCP (DC01)

### 6.1 Complete DHCP post-install and authorize (if required)
1) Server Manager - Notifications:
   - If "Complete DHCP configuration" is present, open it.
2) Complete the wizard using domain credentials to authorize DHCP.
3) Verify in DHCP console that the server is authorized and IPv4 node is available.

### 6.2 Create scope and set scope options
1) DHCP console - IPv4 - New Scope.
2) Name: ORION-LAN-10.70.10.0_24
3) Range:
   - Start: 10.70.10.100
   - End: 10.70.10.199
4) Router (Option 003):
   - Do not configure
5) DNS:
   - Domain: orion.lab
   - DNS server: 10.70.10.10
6) Activate scope.

Evidence:
- `./evidence/lab-01_08_dc01_dhcp_scope_options.png`
- `./evidence/lab-01_23_dc01_dhcp_scope_detail.txt`
- `./evidence/lab-01_32_dc01_dhcp_scope_option_003_router.txt`
- `./evidence/lab-01_33_dc01_dhcp_scope_option_006_dns.txt`
- `./evidence/lab-01_34_dc01_dhcp_scope_option_015_domain.txt`
- `./evidence/lab-01_30_dc01_dhcp_scope_optionvalue_netsh.txt`
- `./evidence/lab-01_31_dc01_dhcp_server_optionvalue_netsh.txt`

### 6.3 Create DHCP reservations
1) DHCP console - Scope - Reservations - New Reservation.
2) Add:
   - ORION-L-W11-01 -> 10.70.10.100 (use W11-01 MAC)
   - ORION-L-W11-02 -> 10.70.10.101 (use W11-02 MAC)

Evidence:
- `./evidence/lab-01_22_dc01_dhcp_reservations.txt`

### 6.4 DHCP DNS update behavior (baseline)
1) DHCP console - IPv4 - right-click the DHCP server - Properties - DNS tab.
2) Configure to match baseline behavior:
   - Dynamic updates: On client request
   - Enable updates for older clients: enabled
   - Discard A and PTR records when lease is deleted: enabled
   - PTR updates: enabled
   - Name protection: disabled

Evidence:
- `./evidence/lab-01_38_dc01_dhcp_dns_setting.txt`

---

## 7) Create OU structure (DC01)

1) Open Active Directory Users and Computers.
2) Under domain `orion.lab`, create OU `Corp`.
3) Under `Corp`, create:
   - Admins
   - Groups
   - Servers
   - Users
   - Workstations

Evidence:
- `./evidence/lab-01_04_dc01_aduc_ou_structure.png`

---

## 8) Build and join Windows 11 clients

### 8.1 W11-01 and W11-02 OS install
- Install Windows 11 Pro on both VMs.
- Ensure NIC is attached to VMnet10 (LAN-LAB) and uses DHCP.

### 8.2 Rename and join domain
For each workstation:
1) Rename:
   - ORION-L-W11-01
   - ORION-L-W11-02
2) Join domain:
   - orion.lab
3) Reboot.

Evidence:
- `./evidence/lab-01_11_w11_01_domain_joined.png`
- `./evidence/lab-01_12_w11_02_domain_joined.png`

### 8.3 Verify client networking and DC discovery
On each workstation (PowerShell):
- `ipconfig /all`
- `nltest /dsgetdc:orion.lab`

Evidence:
- W11-01:
  - `./evidence/lab-01_25_w11_01_ipconfig_all.txt`
  - `./evidence/lab-01_26_w11_01_nltest_dsgetdc_orion_lab.txt`
- W11-02:
  - `./evidence/lab-01_27_w11_02_ipconfig_all.txt`
  - `./evidence/lab-01_28_w11_02_nltest_dsgetdc_orion_lab.txt`

Expected:
- W11-01 IPv4: 10.70.10.100, DNS: 10.70.10.10, no gateway
- W11-02 IPv4: 10.70.10.101, DNS: 10.70.10.10, no gateway
- nltest returns ORION-L-DC01 as the DC

---

## 9) Post-join DNS and DHCP validation (DC01)

### 9.1 Verify W11 A and PTR records exist
1) DNS Manager - Forward Lookup Zones - orion.lab:
   - ORION-L-W11-01 -> 10.70.10.100
   - ORION-L-W11-02 -> 10.70.10.101
2) DNS Manager - Reverse Lookup Zones - 10.70.10.in-addr.arpa:
   - 10.70.10.100 PTR for ORION-L-W11-01
   - 10.70.10.101 PTR for ORION-L-W11-02

Evidence:
- `./evidence/lab-01_05_dc01_dns_forward_zone.png`
- `./evidence/lab-01_06_dc01_dns_reverse_zone.png`
- `./evidence/lab-01_24_dc01_dns_zones_list.txt`

### 9.2 Confirm active leases (reserved addresses)
1) DHCP console - Scope - Address Leases.
2) Confirm leases exist for:
   - 10.70.10.100
   - 10.70.10.101

Evidence:
- `./evidence/lab-01_09_dc01_dhcp_leases.png`

---

## 10) DC health checks (DC01)

### 10.1 SYSVOL and NETLOGON shares
Run:
- `net share`

Evidence:
- `./evidence/lab-01_10_dc01_netshare_sysvol_netlogon.png`
- `./evidence/lab-01_15_dc01_netshare_sysvol_netlogon.txt`
- `./evidence/lab-01_41_dc01_netshare_postcheck.txt`

### 10.2 DCDIAG
Run:
- `dcdiag /v`
- `dcdiag /test:dns /v`
- `dcdiag /c /v`

Evidence:
- `./evidence/lab-01_16_dc01_dcdiag_verbose.txt`
- `./evidence/lab-01_17_dc01_dcdiag_dns_verbose.txt`
- `./evidence/lab-01_35_dc01_dcdiag_full_verbose.txt`

### 10.3 Replication state (single-DC lab)
Run:
- `Get-ADReplicationPartnerMetadata -Target "ORION-L-DC01.orion.lab" -Scope Server | Measure-Object`
- `Get-ADReplicationFailure -Target "ORION-L-DC01.orion.lab" -Scope Server | Measure-Object`

Evidence:
- `./evidence/lab-01_18_dc01_replication_partner_metadata_count.txt`
- `./evidence/lab-01_19_dc01_replication_failure_count.txt`

### 10.4 Time service
Run:
- `w32tm /query /configuration`
- `w32tm /query /status`
- `w32tm /query /source`

Evidence:
- `./evidence/lab-01_36_dc01_w32tm_configuration.txt`
- `./evidence/lab-01_37_dc01_w32tm_status.txt`
- `./evidence/lab-01_43_dc01_w32tm_source.txt`

### 10.5 DFSR and Repadmin
Run:
- DFS Replication event log capture (last 30)
- `repadmin /queue`

Evidence:
- `./evidence/lab-01_42_dc01_dfsr_eventlog_last30.txt`
- `./evidence/lab-01_44_dc01_repadmin_queue.txt`

---

## 11) Completion checklist

### Configuration
- LAN-LAB host-only, 10.70.10.0/24, VMware DHCP disabled
- DC01 static IP 10.70.10.10, DNS self, no default gateway
- New forest orion.lab created, DNS operational
- Reverse zone 10.70.10.in-addr.arpa created, secure updates enabled
- DHCP scope 10.70.10.100-199 created, reservations 100 and 101 configured
- DHCP scope options: 003 not configured, 006 and 015 configured
- W11-01 and W11-02 joined to orion.lab

### Evidence
- Evidence files 01-44 present under `./evidence/`
- TXT evidence is UTF-8
- Commands used are documented in `./commands/commands.md`
