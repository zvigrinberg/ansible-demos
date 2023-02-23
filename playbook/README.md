# Basic Ansible Demo

## Goal - To Show basic usage of ansible, working on existing hosts, install on them some program.

## Objectives
- Run a simple playbook consist out of 2 plays
- One Play will run against set of tasks tailored for Fedora CoreOS Hosts, and the second play will be with set of tasks appropriate for Ubuntu Hosts.  
- Each play will choose the appropriate group of hosts from inventory file.

### Files involved

#### Inventory.yaml
```yaml
virtualMachinesCoreOs:
  hosts:      
   coreos-01:
     ansible_host: 192.168.122.52
   coreos-02:
     ansible_host: 192.168.122.64
   coreos-03:
     ansible_host: 192.168.122.95

  vars:
    successMessage: "installed tree binary on Fedora CoreOS host"

virtualMachinesUbuntu:
  hosts:
    ubuntu-01:
      ansible_host: 192.168.122.233
    ubuntu-02:
      ansible_host: 192.168.122.85
    ubuntu-03:
      ansible_host: 192.168.122.245

  vars:
    successMessage: "installed tree binary on  Ubuntu host"

all:    
  vars:
    key1: "value1"
    key2: "value2" 
```
#### playbook.yaml
```yaml
- name: Install tree fs utility on CoreOs machines and reboot system
  hosts: virtualMachinesCoreOs
  become: true
  tasks:
    - name: Install tree fs utility
      community.general.rpm_ostree_pkg:
        name: tree
        state: present

    - name: reboot the machine
      ansible.builtin.reboot:

    - name: Print success Message
      ansible.builtin.debug:
        #       var: ansible_facts
        msg: "Successfully {{ successMessage }} - {{ ansible_nodename }}, with IP Address: {{ ansible_facts['all_ipv4_addresses'] }} "

- name: Install tree fs utility on ubuntu machines
  hosts: virtualMachinesUbuntu
  become: true
  tasks:
    - name: Install tree fs utility
      ansible.builtin.apt:
        name: tree
        state: present
    - name: Print success Message
      ansible.builtin.debug:
        #        var: ansible_facts
        msg: "Successfully {{ successMessage }} - {{ ansible_nodename }}, with IP Address: {{ ansible_facts['all_ipv4_addresses'] }}  "
```

### Procedure

1. Run the playbook with the inventory above
```shell
ansible-playbook -i inventory.yaml playbook.yaml
```
2. Verify while playbook is running, that at least one Fedora CoreOs is restarting.
3. After playbook run finished, verify on each host that tree utility was installed 