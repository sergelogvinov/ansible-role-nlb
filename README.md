# Ansible role to create Network load balancer

Simple load balancer based on keepalived + ipvs.

This load balancer is versatile and can be used in both home labs and in conjunction with external load balancers.
For instance, it can be used on a Proxmox machine to route requests to various virtual machines.

The load balancer verifies the backends and directs traffic to the healthy backends.
Like many well-known cloud providers, this network load balancer (NLB) has a limited number of options available.
These options include direct routing or destination NATed IP.

## Install

```shell
ansible-galaxy role install git+https://github.com/sergelogvinov/ansible-role-nlb.git,main
```

## Usage

```ini
# inventory file

[servers]
server-1          ansible_host=1.2.3.4
```


```yaml
# hosts/server-1.yaml

nlb_forward_ip:
  - "{{ ansible_default_ipv4['address'] }}"

nlb_forward:
  ingress-http:
    port: 80
    # type: NAT|DR
    # algo: rr|lc|sh|dh
    # proto: TCP|UDP
    backends:
      - { ip: 172.16.0.11, port: 80,  health_check: "http" }
      - { ip: 172.16.0.12, port: 80,  health_check: "http" }
  ingress-https:
    port: 443
    backends:
      - { ip: 172.16.0.11, port: 443, health_check: "https" }
      - { ip: 172.16.0.12, port: 443, health_check: "https" }
```

```yaml
# values.yaml

- hosts: servers
  roles:
    - ansible-role-nlb
```

### Proxmox

You need to add `nf_conntrack_allow_invalid` to the proxmox host firewall.

```ini
; /etc/pve/nodes/$HOST/host.fw

[OPTIONS]
nf_conntrack_allow_invalid: 1
```
