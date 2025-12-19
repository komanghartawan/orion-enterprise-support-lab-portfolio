# LAB-01: Core Identity Foundation (AD DS + DNS + DHCP)

## Objective
Build a clean Windows Server domain foundation with integrated DNS and DHCP, and join two Windows 11 clients to the domain.

## Environment
- Domain: `orion.lab`
- DC: `ORION-L-DC01` (IPv4: `10.70.10.10/24`, no default gateway on LAN-LAB)
- Clients: `ORION-L-W11-01`, `ORION-L-W11-02` (DHCP)
- Network: VMware Host-only `LAN-LAB (VMnet10)` -> `10.70.10.0/24`

## What was implemented
- AD DS forest/domain deployment
- DNS forward and reverse lookup zones
- DHCP scope with options (DNS server + DNS suffix)
- Enterprise-style OU structure under `Corp`
- Clients joined to domain and validated

## Verification checklist (must pass)
- [ ] DC has static IP `10.70.10.10/24` and DNS points to itself
- [ ] `nslookup orion.lab 10.70.10.10` returns A record correctly
- [ ] Reverse lookup zone exists and PTR records are present
- [ ] DHCP scope options set:
  - Option 006 DNS Servers = `10.70.10.10`
  - Option 015 DNS Domain Name = `orion.lab`
- [ ] SYSVOL and NETLOGON shares exist
- [ ] W11-01 and W11-02 joined to domain `orion.lab`

## Evidence pack
| ID | Evidence | File |
|---:|---|---|
| 01 | Host LAN-LAB (VMnet10) subnet | `evidence/LAB-01_01_HOST_LAN-LAB.png` |
| 02 | DC01 ipconfig /all | `evidence/LAB-01_02_DC01_ipconfig-all.png` |
| 03 | DC01 nslookup orion.lab | `evidence/LAB-01_03_DC01_nslookup-orion.lab.png` |
| 04 | ADUC OU structure | `evidence/LAB-01_04_DC01_ADUC_OU-Structure.png` |
| 05 | DNS forward + reverse | `evidence/LAB-01_05_DC01_DNS_Forward+Reverse.png` |
| 06 | DHCP scope + options | `evidence/LAB-01_06_DC01_DHCP_Scope+Options.png` |
| 07 | DHCP leases show clients | `evidence/LAB-01_07_DC01_DHCP_Leases.png` |
| 08 | SYSVOL + NETLOGON share | `evidence/LAB-01_08_DC01_NETSHARE_SYSVOL_NETLOGON.png` |
| 09 | W11-01 domain joined | `evidence/LAB-01_09_W11-01_DomainJoined.png` |
| 10 | W11-02 domain joined | `evidence/LAB-01_10_W11-02_DomainJoined.png` |

## Runbook
See: [runbook.md](runbook.md)