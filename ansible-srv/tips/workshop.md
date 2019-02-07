Create : `~/pb.juniper.bgp-check.yaml`

```
---
- name: Check BGP
  hosts: all
  connection: local
  gather_facts: no

  roles:
    - Juniper.junos

  vars:
    ansible_network_os: junos
    credentials:
      username: noc
      ssh_keyfile: ~/.ssh/id_rsa
      host: "{{ ansible_host }}"

  tasks:

    - name: check bgp peers states
      junos_command:
       provider: "{{  credentials }}"
       commands:
        - show bgp neighbor "{{ item.neighbor_ipv4 }}"
       display: 'xml'
       waitfor:
       - "result[0]['rpc-reply']['bgp-information']['bgp-peer']['peer-state'] == Established"
       retries: 15
       interval: 2
      with_items:
         - "{{ peers }}"
```

Add the below lines to the file : `all/bgp_filter.yaml`

```
---
filtered_prefixes:
  65500:
    ipv4:
    - 2.0.0.0/8
```

