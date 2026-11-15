# Security Policy and ACLs

Extended named ACLs are applied **inbound** on each user-VLAN sub-interface (closest to the source). VLAN 40 (servers) has no ACL, so server replies are not filtered.

| Policy | Rule |
|--------|------|
| A | All staff VLANs may use HTTP (80) and HTTPS (443) to the web server 192.168.40.10, and DNS (53) to it |
| B | Only IT Staff may use FTP (21/20) to 192.168.40.30 and SSH/Telnet; Teller and Manager are denied FTP, SSH and Telnet |
| C | Everything else is denied (`deny ip any any` at the end of every ACL) |

## Per-VLAN summary

| Service | TELLER_ACL (10) | MANAGER_ACL (20) | IT_ACL (30) |
|---------|:---:|:---:|:---:|
| DNS to 192.168.40.10 | permit | permit | permit |
| HTTP/HTTPS to 192.168.40.10 | permit | permit | permit |
| SMTP/POP3 to 192.168.40.20 | permit | permit | permit |
| FTP to 192.168.40.30 | deny | deny | permit (21, 20) |
| SSH / Telnet | deny | deny | permit (to any) |
| ICMP | gateway only | gateway only | any |
| Other VLANs | deny | deny | deny (except above) |

The full rule sets are in `configs/running/R-BANK.running-config.txt`.

## Defense in depth

Teller and Manager accounts are intentionally not created on the FTP server, so FTP is denied at the network layer (ACL) and at the application layer (no credentials).
