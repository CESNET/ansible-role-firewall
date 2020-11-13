cesnet.firewall
======================

Ansible Galaxy role [cesnet.firewall](https://galaxy.ansible.com/cesnet/firewall) 
that installs firewall (iptables or nftables). The iptables version allows rules for Docker.

Use "--tags config" to run only config.

The role creates a systemd service named "iptables" for applying rules and enables it for reboots.
Apply rules from /etc/iptables with:
```bash
systemctl restart iptables
```

Role Variables
--------------
- firewall_open_ssh_ports - predefined rules for accepting ssh only from networks of MUNI,CESNET,ZCU 
- firewall_open_tcp_ports - empty set, define as in the example below 
- firewall_known_ranges - known ranges for the Czech academic network
- firewall_docker_rules - empty set, one rule per port can be defined for restricting access to Docker containers

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
          - { port: 9000, ipv6: "2001:718:801::/48", comment: "accept portainer from MUNI" }
        firewall_docker_rules:
          - { port: 9000, only: "147.251.0.0/16", comment: "portainer only from MUNI" }
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

Docker compatibility
------
TCP ports exported from Docker containers are exposed in the FORWARD chain which
is processed before INPUT chain, so the firewall rules from the INPUT chain do not apply to containers,
see [Docker and iptables](https://docs.docker.com/network/iptables/).

You can put rules into the chain DOCKER-USER for rejecting packets before they reach the chain DOCKER.
Thus there are only two options for a port exported from a container:
* the port is exposed globally
* packets from only a single network can be allowed, all others are rejected

Docker manipulates only IPv4 iptables. Ports exported from containers do listen on all IPv6 addresses, so
rules from the INPUT chain do apply to IPv6 packets. Thus you have to explicitly allow a port exported
from a container to be available over IPv6.

In short, if you want to restrict access to a port 9000 exported from a container in both IPv4 and IPv6,
do it like this:
```yaml
    - role: cesnet.firewall
      vars:
        firewall_open_tcp_ports:
          - { port: 9000, ipv6: "2001:718:801::/48", comment: "accept 9000 only from MUNI over IPv6" }
        firewall_docker_rules:
          - { port: 9000, only: "147.251.0.0/16", comment: "accept 9000 only from MUNI over IPv4" }
```