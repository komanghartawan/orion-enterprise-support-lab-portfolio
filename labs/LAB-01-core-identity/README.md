# LAB-01 Core Identity

## Overview
LAB-01 establishes the identity baseline for the ORION Enterprise Support Lab Portfolio: a single-domain Active Directory environment with DNS and DHCP centralized on one Domain Controller, plus two Windows 11 clients joined to the domain.

The lab is intentionally isolated (host-only) to keep behavior deterministic and reproducible.

## Architecture
- VMware network: Host-only `LAN-LAB` (VMnet10)
- Subnet: `10.70.10.0/24`
- Default gateway: none
- VMware DHCP: Disabled
- Domain: `orion.lab` (NetBIOS: `ORION`)

### Systems
- `ORION-L-DC01` (Windows Server 2025)
  - IPv4: `10.70.10.10/24` (static)
  - DNS: `10.70.10.10`
  - Roles: AD DS, DNS, DHCP
- `ORION-L-W11-01` (Windows 11 Pro)
  - DHCP reservation: `10.70.10.100`
- `ORION-L-W11-02` (Windows 11 Pro)
  - DHCP reservation: `10.70.10.101`

### DHCP baseline (DC01)
- Scope: `10.70.10.100-10.70.10.199`
- Scope options:
  - 003 Router: not configured
  - 006 DNS Servers: `10.70.10.10`
  - 015 DNS Domain Name: `orion.lab`

## Repository layout
- `runbook.md` - step-by-step build procedure
- `commands/commands.md` - commands used to generate evidence as UTF-8 raw output
- `diagrams/` - architecture diagram(s)
- `evidence/` - authoritative proof artifacts (PNG and TXT)

## Reproduce this lab
1) Follow `runbook.md` to build the environment.
2) Use `commands/commands.md` to capture evidence outputs as UTF-8 text (raw output, no commentary).
3) Store artifacts under `evidence/` using the existing naming convention.

## Diagram
- `./diagrams/lab-01_architecture.png`

## Evidence index
All items below are stored under `./evidence/`.

| ID | Evidence file | What it shows |
|---:|---|---|
| 01 | `lab-01_01_host_vmnet10_lan_lab.png` | VMware Virtual Network Editor view for `LAN-LAB` (VMnet10) configuration |
| 02 | `lab-01_02_dc01_ipconfig_all.png` | DC01 `ipconfig /all` (screen capture) |
| 03 | `lab-01_03_dc01_nslookup_orion_lab.png` | DC01 `nslookup` resolution for `orion.lab` (screen capture) |
| 04 | `lab-01_04_dc01_aduc_ou_structure.png` | ADUC view showing the OU structure under `orion.lab` |
| 05 | `lab-01_05_dc01_dns_forward_zone.png` | DNS Manager view of the `orion.lab` forward lookup zone |
| 06 | `lab-01_06_dc01_dns_reverse_zone.png` | DNS Manager view of the reverse lookup zone (`10.70.10.in-addr.arpa`) |
| 07 | `lab-01_07_dc01_dns_reverse_zone_properties.png` | Reverse zone properties view (AD-integrated and dynamic update policy at capture time) |
| 08 | `lab-01_08_dc01_dhcp_scope_options.png` | DHCP scope options view (Option 006 and 015 visible at capture time) |
| 09 | `lab-01_09_dc01_dhcp_leases.png` | DHCP address leases view (reservations and lease state at capture time) |
| 10 | `lab-01_10_dc01_netshare_sysvol_netlogon.png` | `net share` output (screen capture) showing SYSVOL and NETLOGON shares |
| 11 | `lab-01_11_w11_01_domain_joined.png` | System Properties view showing W11-01 joined to `orion.lab` |
| 12 | `lab-01_12_w11_02_domain_joined.png` | System Properties view showing W11-02 joined to `orion.lab` |
| 13 | `lab-01_13_dc01_ipconfig_all.txt` | DC01 `ipconfig /all` (raw output) |
| 14 | `lab-01_14_dc01_nslookup_orion_lab.txt` | DC01 `nslookup orion.lab 10.70.10.10` (raw output) |
| 15 | `lab-01_15_dc01_netshare_sysvol_netlogon.txt` | DC01 `net share` (raw output) |
| 16 | `lab-01_16_dc01_dcdiag_verbose.txt` | DC01 `dcdiag /v` (raw output) |
| 17 | `lab-01_17_dc01_dcdiag_dns_verbose.txt` | DC01 `dcdiag /test:dns /v` (raw output) |
| 18 | `lab-01_18_dc01_replication_partner_metadata_count.txt` | `Get-ADReplicationPartnerMetadata ... | Measure-Object` output (raw) |
| 19 | `lab-01_19_dc01_replication_failure_count.txt` | `Get-ADReplicationFailure ... | Measure-Object` output (raw) |
| 20 | `lab-01_20_dc01_get_addomain.txt` | `Get-ADDomain` (raw output) |
| 21 | `lab-01_21_dc01_get_adforest.txt` | `Get-ADForest` (raw output) |
| 22 | `lab-01_22_dc01_dhcp_reservations.txt` | DHCP reservations list (raw output) |
| 23 | `lab-01_23_dc01_dhcp_scope_detail.txt` | DHCP scope details (raw output) |
| 24 | `lab-01_24_dc01_dns_zones_list.txt` | DNS zones inventory (raw output) |
| 25 | `lab-01_25_w11_01_ipconfig_all.txt` | W11-01 `ipconfig /all` (raw output) |
| 26 | `lab-01_26_w11_01_nltest_dsgetdc_orion_lab.txt` | W11-01 `nltest /dsgetdc:orion.lab` (raw output) |
| 27 | `lab-01_27_w11_02_ipconfig_all.txt` | W11-02 `ipconfig /all` (raw output) |
| 28 | `lab-01_28_w11_02_nltest_dsgetdc_orion_lab.txt` | W11-02 `nltest /dsgetdc:orion.lab` (raw output) |
| 29 | `lab-01_29_dc01_get_addomaincontroller.txt` | `Get-ADDomainController` output for the domain (raw) |
| 30 | `lab-01_30_dc01_dhcp_scope_optionvalue_netsh.txt` | DHCP scope option values via `netsh` (raw output) |
| 31 | `lab-01_31_dc01_dhcp_server_optionvalue_netsh.txt` | DHCP server option values via `netsh` (raw output) |
| 32 | `lab-01_32_dc01_dhcp_scope_option_003_router.txt` | DHCP option 003 query output (raw) |
| 33 | `lab-01_33_dc01_dhcp_scope_option_006_dns.txt` | DHCP option 006 query output (raw) |
| 34 | `lab-01_34_dc01_dhcp_scope_option_015_domain.txt` | DHCP option 015 query output (raw) |
| 35 | `lab-01_35_dc01_dcdiag_full_verbose.txt` | DC01 `dcdiag /c /v` (raw output) |
| 36 | `lab-01_36_dc01_w32tm_configuration.txt` | `w32tm /query /configuration` (raw output) |
| 37 | `lab-01_37_dc01_w32tm_status.txt` | `w32tm /query /status` (raw output) |
| 38 | `lab-01_38_dc01_dhcp_dns_setting.txt` | DHCP DNS update settings capture (raw output) |
| 39 | `lab-01_39_dc01_operation_master_roles.txt` | FSMO roles query output (raw) |
| 40 | `lab-01_40_dc01_ad_partitions.txt` | AD partitions query output (raw) |
| 41 | `lab-01_41_dc01_netshare_postcheck.txt` | `net share` post-check capture (raw output) |
| 42 | `lab-01_42_dc01_dfsr_eventlog_last30.txt` | DFS Replication event log export (raw output) |
| 43 | `lab-01_43_dc01_w32tm_source.txt` | `w32tm /query /source` (raw output) |
| 44 | `lab-01_44_dc01_repadmin_queue.txt` | `repadmin /queue` (raw output) |

## Notes
- This lab uses no default gateway to prevent unintended outbound connectivity and to keep troubleshooting deterministic.
- Evidence TXT files are stored as UTF-8 raw output to preserve auditability.
