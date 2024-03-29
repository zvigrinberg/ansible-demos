# ansible-demos

## Objectives:
- Create a Sample Task, Basically will not needed as most of the tasks are already ready to use and invoking modules code.
- Create a sample playbook
- Create a sample role
- Create a sample inventory
- Create in advance 7 VMs for demos

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
13. Login With user and password you've defined in installation process.
14. Get the IP Address of the machine and write it down
```shell
ip a | grep enp1s0 | grep inet | awk '{print $2}' | awk -F / '{print $1}
```
15. Type exit, and then `Ctrl + ]` In order to quit from emulation
16. Verify that you can ssh into VM from host without password:
```shell
ssh zgrinber@ip_from_section14
```

### Create CoreOs Virtual Machines:

1. Run the command, and input a password to be hashed, this will be the user that will be privileged and will have sudo access on VM:
```shell
podman run -ti --rm quay.io/coreos/mkpasswd --method=yescrypt
```
2. Print one of the public ssh keys in /home/zgrinber/.ssh/, for example:
```shell
cat ~/.ssh/id_ed25519.pub
```
3. Create a butane configuration file with your desired ssh public key from section 2 and hashed password from section 1:
```yaml
variant: fcos
version: 1.4.0
passwd:
  users:
    - name: zgrinber # User name that will have sudo access
      groups:
        - wheel
        - sudo
        - docker
      password_hash: hashed_password # Output From Section 1
      ssh_authorized_keys:
        - your_public_key_content # Output From Section 2


storage:
  files:
    - path: /etc/hostname
      mode: 0644
      contents:
        inline: core-os-1 # Host name

```
4. Create an ignition file (JSON) from Butane YAML:
```shell
podman run --interactive --rm quay.io/coreos/butane:release \
       --pretty --strict < coreos-vm.bu > coreos-vm.ign
```
5. Download fedora CoreOS latest QEMU image:
```shell
curl -X GET https://builds.coreos.fedoraproject.org/prod/streams/stable/builds/37.20221225.3.0/x86_64/fedora-coreos-37.20221225.3.0-qemu.x86_64.qcow2.xz -o /tmp/fedora-coreos-37.20221225.3.0-qemu.x86_64.qcow2.xz
```
5. Uncompress it to same directory
```shell
xz -k -d /tmp/fedora-coreos-37.20221225.3.0-qemu.x86_64.qcow2.xz
```
6. Set the following environment variables for the current terminal session:
```shell
echo "IGNITION_CONFIG="\"$(pwd)\"/coreos-vm.ign"
IMAGE=\"/tmp/fedora-coreos/fedora-coreos-37.20221127.3.0-qemu.x86_64.qcow2\"
VM_NAME=\"coreos-01\"
VCPUS=\"2\"
RAM_MB=\"2048\"
STREAM=\"stable\"
DISK_GB=\"10\"" | tee vm-args.properties ; source vm-args.properties
```

7. Define the following environment variable, using ignition file path in its environment variables:
```shell
export IGNITION_DEVICE_ARG=(--qemu-commandline="-fw_cfg name=opt/com.coreos/config,file=${IGNITION_CONFIG}")
```

8. Provision virtual machine using ignition file and environment variables defined earlier
```shell
sudo virt-install --connect="qemu:///system" --name="${VM_NAME}" --vcpus="${VCPUS}" --memory="${RAM_MB}"         --os-variant="fedora-coreos-$STREAM" --import --graphics=none         --disk="size=${DISK_GB},backing_store=${IMAGE}"         --network bridge=virbr0 "${IGNITION_DEVICE_ARG[@]}"
```
9. After CoreOs VM Startup, login using the credentials you've set in ignition file.
10. Install Python 3 using RPM package manager
```shell
sudo rpm-ostree install python3
```
11. Get the IP Address of the machine and write it down
```shell
ip a | grep enp1s0 | grep inet | awk '{print $2}' | awk -F / '{print $1}
```
12. Reboot machine
```shell
sudo systemctl reboot
```
13. Type exit, and then `Ctrl + ]` In order to quit from emulation
14. Verify that you can ssh into VM from host without password:
```shell
ssh zgrinber@ip_from_section11
```
15. Repeat steps 8-14 for each additional CoreOS VM, with these 2 additional steps To change machine name before 8-14 steps:
```shell
# For example, for provisioning another Fedore CoreOs Machine with name coreos-02, run the following, and then repeat steps 8-14
export VM_NAME=coreos-02 ; sed -i -E 's/source\": \"data:,core-os-[0-9x]{1,2}\"/source\": \"data:,'$VM_NAME'/g' coreos-vm.ign
```
   
### Advanced Demo 

#### Goal

To Create Kubernetes Cluster of 1 master and 2 workers using Ansible and LXC (Linux Containers Virtualization Technology) on Ubuntu Host.


#### Prerequisites

- Ubuntu Host ( Tested on version 20.04) with root/sudo privileges.
- At Least free 50 GB disk space on Host. 
- Ansible And Python Installed on Host.
- Lxc Installed On Host
- Host with free 6 VCPUs and 8 GB RAM.


#### Manual Procedure.

1. SSH into ubuntu HOST.
2. Install LXC 
```shell
sudo apt-get update && sudo apt-get install lxc -y
```
3. Check that lxc service is up and running, and init lxd
```shell
sudo systemctl status lxc
lxd init
```
4. Create empty LXC profile for k8s:
```shell
lxc profile create k8s
cat > ./advanced-demo/k8s-config-profile.j2 << EOF
config:
  limits.cpu: "2"
  limits.memory: 2GB
  limits.memory.swap: "false"
  linux.kernel_modules: ip_tables,ip6_tables,nf_nat,overlay,br_netfilter
  raw.lxc: "lxc.apparmor.profile=unconfined\nlxc.cap.drop= \nlxc.cgroup.devices.allow=a\nlxc.mount.auto=proc:rw
    sys:rw\nlxc.mount.entry = /dev/kmsg dev/kmsg none defaults,bind,create=file"
  security.privileged: "true"
  security.nesting: "true"
description: LXD profile for Kubernetes
devices:
  eth0:
    name: eth0
    nictype: bridged
    parent: lxdbr0
    type: nic
  root:
    path: /
    pool: default
    type: disk
name: k8s
used_by: []
EOF

cat ./advanced-demo/k8s-config-profile.j2 | lxc profile edit k8s
```

5. Make sure that you can see profile created alongside default:
```shell
lxc profile list
```
6. Create 1 container for master and 2 containers for workers , using the k8s profile:
```shell
lxc launch ubuntu:20.04 kmaster --profile k8s
lxc launch ubuntu:20.04 kworker1 --profile k8s
lxc launch ubuntu:20.04 kworker2 --profile k8s
```

7. If host is using cgroup v1, skip this step (only needed for cgroup v2):
```shell
lxc config device add kmaster "kmsg" unix-char source="/dev/kmsg" path="/dev/kmsg"
lxc config device add kworker1 "kmsg" unix-char source="/dev/kmsg" path="/dev/kmsg"
lxc config device add kworker2 "kmsg" unix-char source="/dev/kmsg" path="/dev/kmsg"
lxc restart kmaster kworker1 kworker2
```
8. Install Kubernetes on master container, and wait for it to finish:
```shell
cat advanced-demo/bootstrap-kube.sh | lxc exec kmaster bash
```

9. Install Kubernetes on two worker containers as worker nodes, and join them to the cluster:
```shell
cat advanced-demo/bootstrap-kube.sh | lxc exec kworker1 bash
cat advanced-demo/bootstrap-kube.sh | lxc exec kworker2 bash
```
10. Check that cluster provisioned correctly:
```shell
lxc exec kmaster bash
kubectl get nodes
```

12. Make cluster accessible from host:
```shell
mkdir -p ~/.kube
lxc file pull kmaster/etc/kubernetes/admin.conf ~/.kube/config
# If kubectl doesn't installed in host, kindly install it 
sudo snap install --classic kubectl
kubectl get nodes
```
