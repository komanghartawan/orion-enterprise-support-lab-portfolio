# LAB-01 Core Identity - Evidence Capture Commands

## Scope
This document lists the capture steps and commands used to generate LAB-01 evidence artifacts stored under:
- labs/LAB-01-core-identity/evidence/

## Lab baseline
- Domain: orion.lab (NetBIOS: ORION)
- VMware network: Host-only LAN-LAB (VMnet10)
- Subnet: 10.70.10.0/24
- Default gateway: none
- VMware DHCP: Disabled
- ORION-L-DC01 (Windows Server 2025), IPv4: 10.70.10.10/24 (static), DNS: 10.70.10.10
- ORION-L-W11-01 (Windows 11 Pro), DHCP reservation: 10.70.10.100
- ORION-L-W11-02 (Windows 11 Pro), DHCP reservation: 10.70.10.101
- DHCP scope: 10.70.10.100-199
- DHCP options:
  - 003 Router = (not configured)
  - 006 DNS Servers = 10.70.10.10
  - 015 DNS Domain Name = orion.lab

## Conventions

### File naming
- Pattern: lab-01_nn_system_what.ext
- Lowercase only, underscores only.
- nn is two digits. Do not renumber existing files.
- Append new evidence using the next available nn.

### Output locations
Capture evidence locally on the machine where commands are executed, then copy the resulting files into:
- labs/LAB-01-core-identity/evidence/

Recommended local capture folder (create once per machine):
  $cap = "C:\orion-evidence\LAB-01"
  New-Item -ItemType Directory -Force -Path $cap | Out-Null

### TXT output encoding
- All TXT evidence must be UTF-8.
- Use Out-File with UTF-8 and wide output to avoid wrapped lines.
- For classic tools (ipconfig, nslookup, netdom, repadmin, dcdiag, netsh), use cmd /c.
- Run these commands in PowerShell (Windows PowerShell is sufficient).

Standard patterns:
  cmd /c "<command>" | Out-File -Encoding utf8 -Width 4096 "<outputfile>"
  <cmdlet> | Out-File -Encoding utf8 -Width 4096 "<outputfile>"

PowerShell note:
- Windows PowerShell 5.1: -Encoding utf8 typically writes UTF-8 with BOM (Notepad-friendly).
- PowerShell 7+: use -Encoding utf8BOM if you require BOM.

### PNG capture
- Use Snipping Tool (or equivalent).
- Crop to the relevant area.
- Avoid exposing unrelated personal identifiers.

## Evidence capture map

### Host (VMware)

#### lab-01_01_host_vmnet10_lan_lab.png
Capture:
- Open VMware Virtual Network Editor
- Select LAN-LAB (VMnet10) Host-only
- Confirm:
  - Subnet IP: 10.70.10.0
  - Subnet mask: 255.255.255.0
  - Use local DHCP service to distribute IP address to VMs: unchecked
  - Connect a host virtual adapter to this network: checked
- Save screenshot as lab-01_01_host_vmnet10_lan_lab.png

### DC01 (ORION-L-DC01) PNG

#### lab-01_02_dc01_ipconfig_all.png
Capture:
- Run: ipconfig /all
- Screenshot showing IPv4 10.70.10.10, gateway blank, DNS 10.70.10.10

#### lab-01_03_dc01_nslookup_orion_lab.png
Capture:
- Run: nslookup orion.lab 10.70.10.10
- Screenshot successful resolution

#### lab-01_04_dc01_aduc_ou_structure.png
Capture:
- Active Directory Users and Computers
- Show OU structure under Corp (Admins, Groups, Servers, Users, Workstations)

#### lab-01_05_dc01_dns_forward_zone.png
Capture:
- DNS Manager > Forward Lookup Zones > orion.lab
- Show A records for ORION-L-DC01, ORION-L-W11-01, ORION-L-W11-02

#### lab-01_06_dc01_dns_reverse_zone.png
Capture:
- DNS Manager > Reverse Lookup Zones > 10.70.10.in-addr.arpa
- Show PTR records for 10, 100, 101

#### lab-01_07_dc01_dns_reverse_zone_properties.png
Capture:
- Reverse zone properties
- Show Type: Active Directory-Integrated and Dynamic updates: Secure only

#### lab-01_08_dc01_dhcp_scope_options.png
Capture:
- DHCP console > Scope [10.70.10.0] > Scope Options
- Show 006 DNS Servers and 015 DNS Domain Name

#### lab-01_09_dc01_dhcp_leases.png
Capture:
- DHCP console > Scope > Address Leases
- Show reservation leases active for 10.70.10.100 and 10.70.10.101

#### lab-01_10_dc01_netshare_sysvol_netlogon.png
Capture:
- Run: net share
- Screenshot showing SYSVOL and NETLOGON

### Workstations PNG

#### lab-01_11_w11_01_domain_joined.png
Capture:
- W11-01: System Properties > Computer Name
- Show Full computer name and Domain: orion.lab

#### lab-01_12_w11_02_domain_joined.png
Capture:
- W11-02: System Properties > Computer Name
- Show Full computer name and Domain: orion.lab

## TXT evidence commands

### DC01 commands (run on ORION-L-DC01)
Create capture folder:
  $cap = "C:\orion-evidence\LAB-01"
  New-Item -ItemType Directory -Force -Path $cap | Out-Null

Commands:
  cmd /c "ipconfig /all" | Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_13_dc01_ipconfig_all.txt")
  cmd /c "nslookup orion.lab 10.70.10.10" | Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_14_dc01_nslookup_orion_lab.txt")
  cmd /c "net share" | Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_15_dc01_netshare_sysvol_netlogon.txt")
  cmd /c "dcdiag /v" | Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_16_dc01_dcdiag_verbose.txt")
  cmd /c "dcdiag /test:dns /v" | Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_17_dc01_dcdiag_dns_verbose.txt")

  Get-ADReplicationPartnerMetadata -Target "ORION-L-DC01.orion.lab" -Scope Server | Measure-Object |
    Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_18_dc01_replication_partner_metadata_count.txt")

  Get-ADReplicationFailure -Target "ORION-L-DC01.orion.lab" -Scope Server | Measure-Object |
    Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_19_dc01_replication_failure_count.txt")

  Get-ADDomain | Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_20_dc01_get_addomain.txt")
  Get-ADForest | Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_21_dc01_get_adforest.txt")

  Get-DhcpServerv4Reservation -ScopeId 10.70.10.0 |
    Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_22_dc01_dhcp_reservations.txt")

  Get-DhcpServerv4Scope -ScopeId 10.70.10.0 |
    Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_23_dc01_dhcp_scope_detail.txt")

  Get-DnsServerZone | Sort-Object ZoneName |
    Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_24_dc01_dns_zones_list.txt")

  Get-ADDomainController -Identity "ORION-L-DC01" |
    Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_29_dc01_get_addomaincontroller.txt")

  cmd /c "netsh dhcp server scope 10.70.10.0 show optionvalue" |
    Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_30_dc01_dhcp_scope_optionvalue_netsh.txt")

  cmd /c "netsh dhcp server show optionvalue" |
    Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_31_dc01_dhcp_server_optionvalue_netsh.txt")

  $opt3 = Get-DhcpServerv4OptionValue -ScopeId 10.70.10.0 -OptionId 3 -ErrorAction SilentlyContinue
  if ($null -eq $opt3) { "OptionId 003 (Router) is NOT configured for scope 10.70.10.0" } else { $opt3 } |
    Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_32_dc01_dhcp_scope_option_003_router.txt")

  Get-DhcpServerv4OptionValue -ScopeId 10.70.10.0 -OptionId 6 |
    Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_33_dc01_dhcp_scope_option_006_dns.txt")

  Get-DhcpServerv4OptionValue -ScopeId 10.70.10.0 -OptionId 15 |
    Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_34_dc01_dhcp_scope_option_015_domain.txt")

  cmd /c "dcdiag /c /v" | Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_35_dc01_dcdiag_full_verbose.txt")

  cmd /c "w32tm /query /configuration" |
    Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_36_dc01_w32tm_configuration.txt")

  cmd /c "w32tm /query /status" |
    Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_37_dc01_w32tm_status.txt")

  Get-DhcpServerv4DnsSetting |
    Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_38_dc01_dhcp_dns_setting.txt")

  cmd /c "netdom query fsmo" |
    Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_39_dc01_operation_master_roles.txt")

  Get-ADPartition | Select-Object DistinguishedName |
    Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_40_dc01_ad_partitions.txt")

  cmd /c "net share" |
    Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_41_dc01_netshare_postcheck.txt")

  Get-WinEvent -LogName "DFS Replication" -MaxEvents 30 |
    Select-Object TimeCreated, Id, LevelDisplayName, Message |
    Format-List |
    Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_42_dc01_dfsr_eventlog_last30.txt")

  cmd /c "w32tm /query /source" |
    Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_43_dc01_w32tm_source.txt")

  cmd /c "repadmin /queue" |
    Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_44_dc01_repadmin_queue.txt")

Copy the generated TXT files from:
- C:\orion-evidence\LAB-01\
into:
- labs/LAB-01-core-identity/evidence/

### W11-01 commands (run on ORION-L-W11-01)
Create capture folder:
  $cap = "C:\orion-evidence\LAB-01"
  New-Item -ItemType Directory -Force -Path $cap | Out-Null

Commands:
  cmd /c "ipconfig /all" | Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_25_w11_01_ipconfig_all.txt")
  cmd /c "nltest /dsgetdc:orion.lab" | Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_26_w11_01_nltest_dsgetdc_orion_lab.txt")

Copy the generated TXT files from:
- C:\orion-evidence\LAB-01\
into:
- labs/LAB-01-core-identity/evidence/

### W11-02 commands (run on ORION-L-W11-02)
Create capture folder:
  $cap = "C:\orion-evidence\LAB-01"
  New-Item -ItemType Directory -Force -Path $cap | Out-Null

Commands:
  cmd /c "ipconfig /all" | Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_27_w11_02_ipconfig_all.txt")
  cmd /c "nltest /dsgetdc:orion.lab" | Out-File -Encoding utf8 -Width 4096 (Join-Path $cap "lab-01_28_w11_02_nltest_dsgetdc_orion_lab.txt")

Copy the generated TXT files from:
- C:\orion-evidence\LAB-01\
into:
- labs/LAB-01-core-identity/evidence/

## Extensions
The current LAB-01 evidence set ends at lab-01_44. If additional captures are needed later, continue numbering using the next available nn and add the corresponding evidence files under:
- labs/LAB-01-core-identity/evidence/
