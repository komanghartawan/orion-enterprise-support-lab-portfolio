# LAB-01 Runbook: AD DS + DNS + DHCP + 2 Clients

## 1. Network baseline (Host)
- VMware Virtual Network Editor:
  - LAN-LAB (VMnet10) Host-only
  - Subnet: 10.70.10.0 / 255.255.255.0
  - VMware DHCP: disabled

## 2. DC01 baseline
- Rename: ORION-L-DC01
- Static IP: 10.70.10.10/24
- Default gateway: none (LAN-LAB)
- DNS: 10.70.10.10

## 3. Install roles
- AD DS
- DNS
- DHCP

## 4. Promote to Domain Controller
- New forest: orion.lab
- Forest functional level: Windows Server 2025
- Domain functional level: Windows Server 2025

## 5. DNS configuration
- Confirm forward zone: orion.lab
- Create reverse zone: 10.70.10.in-addr.arpa

## 6. DHCP configuration
- Scope: 10.70.10.0/24
- Range: (your chosen range)
- Options:
  - 006 DNS Servers = 10.70.10.10
  - 015 DNS Domain Name = orion.lab

## 7. OU structure
- Create OU: Corp
- Under Corp: Admins, Users, Groups, Servers, Workstations

## 8. Clients
- ORION-L-W11-01 and ORION-L-W11-02:
  - NIC: LAN-LAB
  - IP: DHCP
  - Join domain: orion.lab

## 9. Verification
- ipconfig /all on DC01
- nslookup orion.lab 10.70.10.10 on DC01
- DHCP Leases show W11-01 and W11-02
- net share shows SYSVOL and NETLOGON
- System Properties on W11 show domain joined