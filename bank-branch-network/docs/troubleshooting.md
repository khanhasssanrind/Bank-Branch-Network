# Troubleshooting

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| All inter-VLAN pings fail | Router G0/0 is shut down | `interface g0/0` then `no shutdown` |
| VLAN exists but no traffic | Port in wrong VLAN or mode | `show vlan brief`, `show int fa0/X switchport` |
| Sub-interface down/down | Missing `encapsulation dot1Q` or VLAN not allowed on trunk | Check encapsulation and `show interfaces trunk` |
| Everything is permitted | ACL applied outbound or on the wrong sub-interface | `show ip interface g0/0.X` and check the inbound ACL name |
| IT cannot use FTP | Data channel (TCP 20) not permitted | Confirm `permit tcp ... host 192.168.40.30 eq 20` in IT_ACL |
| ACL counters stay at 0 | Traffic not reaching router (Layer 2 issue) | Check trunk, test intra-VLAN first |
| SSH to router fails | No RSA key or domain name | `ip domain-name bank.com` then `crypto key generate rsa` (skip if `show ip ssh` already says SSH Enabled) |
