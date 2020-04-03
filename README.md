cesnet.firewall
======================

Ansible Galaxy role [cesnet.firewall](https://galaxy.ansible.com/cesnet/firewall) 
that installs firewall (iptables or nftables) and fail2ban.

Use "--tags config" to run only config.

Role Variables
--------------
- firewall_open_ssh_ports - predefined rules for accepting ssh only from networks of MUNI,CESNET,ZCU 
- firewall_open_tcp_ports - empty set, define as in the example below 
- firewall_known_ranges - known ranges for the Czech academic network
Example Playbook
----------------
```yaml
- hosts: all
  roles:
    - role: cesnet.firewall
      vars:
        firewall_open_ssh_ports:
          - { port: "ssh", ipv4: "147.251.0.0/16", comment: "accept ssh from MUNI" }
          - { port: "ssh", ipv4: "147.228.0.0/16", comment: "accept ssh from ZCU" }
          - { port: "ssh", ipv4: "147.231.0.0/16", comment: "accept ssh from CAS" }
          - { port: "ssh", ipv4: "160.217.0.0/16", comment: "accept ssh from JCU" }
          - { port: "ssh", ipv4: "78.128.208.0/20", comment: "accept ssh from CESNET" }
          - { port: "ssh", ipv4: "195.113.0.0/16", comment: "accept ssh from CESNET" }
          - { port: "ssh", ipv6: "2001:718::/32", comment: "accept ssh from CESNET provider" } 
        firewall_open_tcp_ports:
          - { port: 25, ipv4: "147.251.0.0/16", comment: "accept SMTP from MUNI" }
          - { port: 80, comment: "accept http" }
          - { port: 443, comment: "accept https" }
          - { port: 636, comment: "accept ldaps" }
          - { port: 5432, ipv6: "2001:718::/32", comment: "accept postgres from CESNET" }
          - { port: 5432, ipv6: "147.251.0.0/16", comment: "accept postgres from MUNI" }
```
For more complex setups, you can use filters, e.g. to open ports 80 and 443 to known IP ranges only:
```yaml
- hosts: all
  vars:
    my_ranges:
      - { ipv4: "147.251.0.0/16", comment: "allow from MUNI" }
      - { ipv6: "2001:718:801::/48", comment: "allow from MUNI" }
      - { ipv4: "147.32.0.0/16", comment: "allow from CVUT" }
      - { ipv4: "147.228.0.0/16", comment: "allow from ZCU" }
    firewall_open_tcp_ports: "{{ (my_ranges|map('combine',{'port':'http'})|list) + (my_ranges|map('combine',{'port':'https'})|list) }}"
  roles:
    - role: cesnet.firewall

```