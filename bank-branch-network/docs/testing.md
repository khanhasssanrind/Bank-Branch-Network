# Testing Guide

All the ways to verify this network, from the bottom layer up. Run them in this order so a failure points at the right layer.

## 1. Layer 1/2: links, VLANs, trunk (switch)

```
SW-BANK# show vlan brief                   ! VLANs 10,20,30,40 exist; Fa0/1-5 in 10, 6-10 in 20, 11-15 in 30, 16-20 in 40
SW-BANK# show interfaces trunk             ! Gi0/1 trunking, allowed 10,20,30,40, native 99
SW-BANK# show interfaces status            ! used ports connected, Fa0/21-24 and Gi0/2 disabled
SW-BANK# show interfaces fa0/1 switchport  ! mode access, correct access VLAN
SW-BANK# show spanning-tree                ! PortFast on access ports, no blocked trunk
SW-BANK# show mac address-table            ! each device MAC learned in its own VLAN
```

Packet Tracer: green triangles on every link in the topology means the port is up.

## 2. Layer 3: router and routing

```
R-Bank# show ip interface brief            ! G0/0 up/up, G0/0.10/.20/.30/.40 up/up
R-Bank# show ip route                      ! four connected /24 networks
R-Bank# show interfaces gigabitEthernet 0/0.10    ! encapsulation 802.1Q Virtual LAN, Vlan ID 10
R-Bank# show running-config | section interface
```

## 3. Host checks (PC > Desktop > Command Prompt)

```
ipconfig                    ! correct IP, mask, gateway
ping <gateway>              ! first hop works
ping 192.168.40.10          ! server reachability (see ACL expectations below)
tracert 192.168.40.10       ! should show the gateway as hop 1
arp -a                      ! gateway MAC learned
nslookup <name>             ! DNS on 192.168.40.10 (needs records in Services > DNS)
```

## 4. Application tests

| Service | How to test | Where |
|---------|-------------|-------|
| Web | Desktop > Web Browser > `http://192.168.40.10` and `https://192.168.40.10` | Teller, Manager, IT |
| DNS | Browser by name, or `nslookup` | All PCs |
| Mail | Desktop > Email: configure `user@bank.com` with server 192.168.40.20, send a mail between users, then Receive | All PCs |
| FTP (CLI) | `ftp 192.168.40.30`, login, then `dir`, `get`, `put`, `delete`, `quit` | IT-PC1 |
| FTP (permissions) | Log in as `monitor` and confirm `put`/`delete` are refused (read/list only) | IT-PC1 |
| SSH | `ssh -l admin 192.168.40.1` (password 123) | IT-PC1 |
| Telnet | `telnet 192.168.40.1` should fail at the router (SSH-only VTY) even though the ACL lets it through | IT-PC1 |

## 5. Security-policy matrix (expected results)

| Test | From | Expected |
|------|------|----------|
| Web page | Teller, Manager, IT | Pass |
| Mail send/receive | Teller, Manager, IT | Pass |
| FTP to .30 | IT | Pass |
| FTP to .30 | Teller, Manager | Fail (ACL deny) |
| SSH to any device | IT | Pass |
| SSH / Telnet | Teller, Manager | Fail |
| `ping 192.168.40.10` | IT | Pass |
| `ping 192.168.40.10` | Teller, Manager | Fail (ICMP only to own gateway) |
| `ping` own gateway | Teller, Manager, IT | Pass |
| `ping` another staff VLAN | Teller, Manager | Fail |
| `ping` Teller or Manager PC | IT | Fail (no rule to that VLAN) |

Also test negatives from a PC that is not allowed, such as FTP to a wrong IP or HTTP to .20 and .30, which must be denied.

## 6. ACL verification on the router

```
R-Bank# show access-lists                       ! per-rule match counters
R-Bank# show access-lists TELLER_ACL
R-Bank# show access-lists MANAGER_ACL
R-Bank# show access-lists IT_ACL
R-Bank# show ip interface g0/0.10 | include access list    ! inbound list: TELLER_ACL
R-Bank# show ip interface g0/0.20 | include access list    ! inbound list: MANAGER_ACL
R-Bank# show ip interface g0/0.30 | include access list    ! inbound list: IT_ACL
R-Bank# clear access-list counters              ! reset, run one test, re-read the counters
```

Method: clear counters, run one test, then check which line's match count rose. A permit rule increments for a pass; the matching deny (or the final `deny ip any any`) increments for a block.

## 7. Packet Tracer tools

- **Simulation mode**: add a filter for HTTP, FTP, SMTP, POP3, DNS, ICMP, then play a test and click each packet to see which device dropped it and at which ACL (`Inbound ACL denied`).
- **Add Simple PDU** (envelope icon): drag from a source PC to a target for a quick pass/fail ping.
- **Add Complex PDU**: pick TCP with a destination port (21, 22, 23, 25, 80, 110, 443) to test each service port without an app.
- **Inspect** (magnifier): view the router ARP table, routing table and each device's MAC table.
- **Realtime vs Simulation**: the first ping often fails on ARP; send it twice before calling it a failure.

## 8. Resilience and config-hygiene checks

- Shut `G0/0` on the router and confirm all inter-VLAN traffic stops, then `no shutdown` and confirm it recovers.
- Move a PC to a port in another VLAN's range and confirm its gateway and permissions change.
- Plug a device into a shutdown port (Fa0/21) and confirm there is no link.
- `copy running-config startup-config`, reload, and confirm configs and ACL bindings survive.
- `show running-config | include password|secret` to confirm no plaintext on the router (switch `service password-encryption` is off).
