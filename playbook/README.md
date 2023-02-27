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
1. Start all virtual machines
```shell
for vm in coreos-01  coreos-02 coreos-03 ubuntu-01 ubuntu-02 ubuntu-03
do virsh --connect qemu:///system start $vm 
done
```
2. Run the playbook with the inventory above and enter become password (installing packages is a privileged action on VMs):
```shell
ansible-playbook -i inventory.yaml playbook.yaml -K
```
3. Verify while playbook is running, that at least one Fedora CoreOs is restarting.
4. After playbook run finished, verify on each host that tree utility was installed
```shell
for ip in $(virsh --connect qemu:///system net-dhcp-leases default | grep -E 'ubuntu|core-os' | awk '{print $5}' | awk -F / '{print $1}')
do 
echo -n hostname=
ssh zgrinber@$ip hostname  
echo ipAddress=$ip
echo "output of tree -a /home command on '$ip' :"
ssh zgrinber@$ip tree -a /home
ssh zgrinber@$ip uname -a
echo
echo
done
```

5. Now uninstall tree utility using the same playbook
```shell
ansible-playbook  -i inventory.yaml playbook.yaml --extra-vars=operation=absent -K
```

6. When finished, shutdown all active VMs in order to evict/free resources on system:
```shell
virsh --connect qemu:///system list | grep -i running | awk '{print $2}' | xargs -i virsh --connect qemu:///system shutdown {}
```
