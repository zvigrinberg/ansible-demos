# ansible-demos

## Objectives:
- Create a Sample Task, Basically will not needed as most of the tasks are already ready to use and invoking modules code.
- Create a sample playbook
- Create a sample role
- Create a sample inventory
- Create in advance 2-3 vm for demos

## Prerequisites

- Install Ansible suite on control Machine, [follow instructions here](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
- Check if your hardware supporting virtualization
```shell
egrep -c '(vmx|svm)' /proc/cpuinfo
```
- Install QEMU-KVM, and libvirt
  1. RHEL, CentOS:
  ```shell
  sudo yum install qemu-kvm qemu-img libvirt virt-install libvirt-client virt-manager
  ```
  2. Fedora, CoreOS:
  ```shell
  sudo dnf install qemu-kvm qemu-img libvirt virt-install libvirt-client virt-manager
  ```
  3. Debian, Ubuntu:
  ```shell
  sudo apt-get update
  sudo apt-get install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils
  ```
- Sudo/root privileges.
## Create Virtual Machines



### Create Ubuntu VM

1. Download the latest ubuntu Image to your machine:
```shell
curl -X GET https://releases.ubuntu.com/22.04.1/ubuntu-22.04.1-live-server-amd64.iso -o /tmp/ubuntu-22.04.1-live-server-amd64.iso
```

2. Create A Virtual Machine (Repeat the process as many times as needed, according to your local resources storage ,memory and cpu cores):
```shell
 sudo virt-install --connect qemu:///system --name ubuntu-xx --os-variant ubuntu22.04 --vcpus 1 --memory 2048 --location /tmp/ubuntu-22.04.1-live-server-amd64.iso,kernel=casper/vmlinuz,initrd=casper/initrd --network bridge=virbr0,model=virtio --disk size=10 --graphics none --extra-args='console=ttyS0,115200n8 --- console=ttyS0,115200n8' --debug
```

3. the VM is being now created, a guided interactive Installer will appear within 30-45 seconds.
  There you choose "Continue in basic mode" --> then "Continue without updating" --> on Langauge selection screen, let it remain Both Layout and Variant= English (US) --> Press Done --> Checkbox the first option(Ubuntu Server)  and press Done.
4. On Next screen, It will automatically create network interface and will assign it an IP Address to interact with it from host and other machines, just press Done.
5. On Proxy Address and Mirror Address, just press twice on Done.
6. Choose the option "Use an entire disk", and press twice on done, and then on continue.
7. Fill details of your full name, Server name (Machine name), username , and pick up password, this password will used also for privileged escalation using sudo for privileged operations on this VM.
8. Check the option `Install OpenSSH server`
9. Import SSH identity = from GitHub, and write down your github user to import public ssh keys from there and install them on machine as authorized keys, `Allow password authentication over SSH` should remain unchecked, press on Done.
10. Click on Yes.
11. Click on Done on page `Featured Server Snaps`.
12. Wait for installation to finish, and then click on button `Reboot Now`.



