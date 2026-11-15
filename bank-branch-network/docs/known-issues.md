# Known Issues and Hardening Ideas

Found while reviewing the exported configs. These are lab shortcuts, not defects in the design.

- **No enable password/secret** on either device. Add `enable secret <value>`.
- **Switch VTY lines use `login` with no password**, so remote access to the switch will not work. Give the switch a management SVI plus `login local` and SSH.
- **`no service password-encryption`** is set on both devices. Enable it or use `secret` everywhere.
- **No ACL on the VLAN 40 sub-interface**, so servers can initiate traffic to any VLAN. Add a server-side ACL if stricter control is required.
- **Telnet is permitted for IT Staff** in IT_ACL. The router only accepts SSH on its VTY lines, but permitting Telnet to other devices is weaker than SSH-only.
- **`deny ip any any` has no `log` keyword**, so denied traffic is only visible in ACL match counters.
- **Router-on-a-stick** is used for clarity. A Layer 3 switch would be preferable for production throughput.
- **Mail and FTP use plaintext protocols** (SMTP/POP3/FTP). Use TLS variants or SFTP outside a lab.
- **VLAN 99** is referenced as the trunk native VLAN. Make sure it exists in the VLAN database (`show vlan brief`); the setup script creates it.
- **Weak lab credentials** (`123` on nearly every account, `cisco/cisco` on FTP) are fine in a simulation but must never be reused on real equipment.
