# Advanced Ansible Demo - Playbook that invokes two roles

## Objectives: 
- To show capabilities of ansible in provisioning infrastructure ( hosts ) by first role.
- Then use them dynamically in the second role to install on provisioned hosts kubernetes cluster 
- We'll use LXD Linux Containers ( System containers) , which are kind of a very lightweight VM, but still give you the full power of a full operation-system like in VM,
  And as system containers, it has 3 benefits over traditional VM and application containers ( docker, podman , k8s) :
   - It can run multiple processes while application containers basically run only one process ( the entry point of image)
   - Full power of a linux OS is available on the system container, while only part of OS capabilities are available on application containers.
   - system containers are using  and sharing the host' kernel whereas VM has its own kernel, this one make them much faster to be provisioned and more lightweight in comparison to VM.

## Prerequisites
- We will use A Ubuntu version 22.04 VM , with 8 VCPUS , 8GB of RAM Memory, and at least 40 GB of disk size.
  For example, to install it using libvirt with qemu-kvm virtualization:
```shell
sudo virt-install --connect qemu:///system --name ubuntu-demo --os-variant ubuntu22.04 --vcpus 8 --memory 8192 --location /tmp/ubuntu-22.04.1-live-server-amd64.iso,kernel=casper/vmlinuz,initrd=casper/initrd --network bridge=virbr0,model=virtio --disk size=40 --graphics none --extra-args='console=ttyS0,115200n8 --- console=ttyS0,115200n8' --debug
```
- Needed to split the storage among filesystem as follows:
     - `/var` file system will need 25 GB
     - `/` root filesystem will have 9 GB
     - `/home` fs will need 2 GB
     - `/boot` fs will need 2 GB
     - 2 GB should not be allocated at all.
  
- Need to Install python and ansible on that host machine.
- Need to deploy ssh public key on that machine that will be authorized key, that 
  from your computer, will be authenticated using its private key counterpart 
- using the key-pair (private on client, and public on host machine), ssh into that machine with sudo privileges.  \
**Note: The Ubunutu installer has an automatic feature that installs automatically your github ssh public keys on the host (after you input your github account)**

## Procedure
1. Start VM. 
```shell
virsh --connect qemu:///system start ubuntu-demo
# wait 10-15 seconds and get the IP of VM
export UBUNTU_DEMO_VM_IP=$(virsh --connect qemu:///system net-dhcp-leases default | grep ubuntu-demo | awk '{print $5}' | awk -F / '{print $1}')
```

    
2. ssh into ubuntu VM host
```shell
ssh zgrinber@$UBUNTU_DEMO_VM_IP
Welcome to Ubuntu 22.04.1 LTS (GNU/Linux 5.15.0-60-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu Feb 23 01:09:55 UTC 2023

  System load:             0.7958984375
  Usage of /home:          25.5% of 1.90GB
  Memory usage:            29%
  Swap usage:              0%
  Processes:               310
  Users logged in:         1
  IPv4 address for enp1s0: 192.168.122.186
  IPv4 address for lxcbr0: 10.0.3.1
  IPv4 address for lxdbr0: 10.136.18.1
  IPv6 address for lxdbr0: fd42:57fd:9dcd:f597::1

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

56 updates can be applied immediately.
To see these additional updates run: apt list --upgradable


Last login: Wed Feb 22 15:48:28 2023 from 192.168.122.1
zgrinber@ubuntu-demo:~$ 

```

3. Clone this repo, and Enter into advanced-demo directory in the repo
```shell
cd ansible-demos/advanced-demo/
```

4. Deploy the cluster by running the following ansible playbook, which consist from 2 plays, each one invokes a different role:
```yaml
- hosts: localhost
  become: true
  remote_user: root
  roles:
    - create-lxc-k8s-nodes


- hosts: all
  remote_user: root
  roles:
    - install-k8s
```

Input your sudo password:
```shell
ansible-playbook create-k8s-cluster.yaml -K
```

5. The process should take approximately 6 minutes, once the playbook run is finished, you can validate that the installation was successful by running the following:
```shell
kubectl get nodes -o wide
```
output:
```shell
NAME       STATUS   ROLES           AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
kmaster    Ready    control-plane   3m   v1.26.1   10.136.18.147   <none>        Ubuntu 22.04.2 LTS   5.15.0-60-generic   cri-o://1.26.1
kworker1   Ready    <none>          3m   v1.26.1   10.136.18.143   <none>        Ubuntu 22.04.2 LTS   5.15.0-60-generic   cri-o://1.26.1
kworker2   Ready    <none>          3m   v1.26.1   10.136.18.100   <none>        Ubuntu 22.04.2 LTS   5.15.0-60-generic   cri-o://1.26.1
kworker3   Ready    <none>          3m   v1.26.1   10.136.18.128   <none>        Ubuntu 22.04.2 LTS   5.15.0-60-generic   cri-o://1.26.1
```
6. After few seconds, list all pods in cluster
```shell
kubectl get pods -A -o wide
```
Output:
```shell
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE   IP              NODE       NOMINATED NODE   READINESS GATES
kube-system   cilium-dvt6d                       1/1     Running   0          3m   10.136.18.143   kworker1   <none>           <none>
kube-system   cilium-jnmsf                       1/1     Running   0          3m   10.136.18.147   kmaster    <none>           <none>
kube-system   cilium-kz5wm                       1/1     Running   0          3m   10.136.18.128   kworker3   <none>           <none>
kube-system   cilium-operator-5c594d7766-nt884   1/1     Running   0          3m   10.136.18.143   kworker1   <none>           <none>
kube-system   cilium-operator-5c594d7766-sf65b   1/1     Running   0          3m   10.136.18.100   kworker2   <none>           <none>
kube-system   cilium-sfh7m                       1/1     Running   0          3m   10.136.18.100   kworker2   <none>           <none>
kube-system   coredns-787d4945fb-4dwlx           1/1     Running   0          3m   10.0.0.62       kworker2   <none>           <none>
kube-system   coredns-787d4945fb-pmw42           1/1     Running   0          3m   10.0.3.207      kworker3   <none>           <none>
kube-system   etcd-kmaster                       1/1     Running   0          3m   10.136.18.147   kmaster    <none>           <none>
kube-system   kube-apiserver-kmaster             1/1     Running   0          3m   10.136.18.147   kmaster    <none>           <none>
kube-system   kube-controller-manager-kmaster    1/1     Running   0          3m   10.136.18.147   kmaster    <none>           <none>
kube-system   kube-proxy-6rm4q                   1/1     Running   0          3m   10.136.18.143   kworker1   <none>           <none>
kube-system   kube-proxy-7ml7f                   1/1     Running   0          3m   10.136.18.100   kworker2   <none>           <none>
kube-system   kube-proxy-l4qn2                   1/1     Running   0          3m   10.136.18.147   kmaster    <none>           <none>
kube-system   kube-proxy-rzznb                   1/1     Running   0          3m   10.136.18.128   kworker3   <none>           <none>
kube-system   kube-scheduler-kmaster             1/1     Running   0          3m   10.136.18.147   kmaster    <none>           <none>
kube-system   metrics-server-789897fb69-kg5fk    1/1     Running   0          3m   10.136.18.128   kworker3   <none>           <none>
```

7. You can list metrics of cpu and memory usage of nodes and pods:
```shell
kubectl top nodes
echo
kubectl top pods -A
```
Output:
```shell
NAME       CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
kmaster    163m         2%     1225Mi          67%       
kworker1   35m          0%     634Mi           35%       
kworker2   35m          0%     625Mi           34%       
kworker3   37m          0%     626Mi           34%       

NAMESPACE     NAME                               CPU(cores)   MEMORY(bytes)   
kube-system   cilium-dvt6d                       6m           115Mi           
kube-system   cilium-jnmsf                       8m           132Mi           
kube-system   cilium-kz5wm                       8m           102Mi           
kube-system   cilium-operator-5c594d7766-nt884   3m           22Mi            
kube-system   cilium-operator-5c594d7766-sf65b   1m           14Mi            
kube-system   cilium-sfh7m                       14m          129Mi           
kube-system   coredns-787d4945fb-4dwlx           2m           11Mi            
kube-system   coredns-787d4945fb-pmw42           3m           14Mi            
kube-system   etcd-kmaster                       37m          48Mi            
kube-system   kube-apiserver-kmaster             61m          343Mi           
kube-system   kube-controller-manager-kmaster    24m          54Mi            
kube-system   kube-proxy-6rm4q                   1m           13Mi            
kube-system   kube-proxy-7ml7f                   1m           41Mi            
kube-system   kube-proxy-l4qn2                   1m           16Mi            
kube-system   kube-proxy-rzznb                   1m           11Mi            
kube-system   kube-scheduler-kmaster             4m           24Mi            
kube-system   metrics-server-789897fb69-kg5fk    5m           15Mi
```

8. Create a new namespace and deploy there hazelcast application:
```shell
kubectl create ns hazelcast; kubectl run hazelcast --image=hazelcast/hazelcast --port=5701 -n hazelcast
```

9. Check that the  app is up and running:
```shell
kubectl get pods -n hazelcast -w
```
Output:
```shell
NAME        READY   STATUS              RESTARTS   AGE
hazelcast   0/1     ContainerCreating   0          3s
hazelcast   1/1     Running             0          16s
```

10. Check logs of the application:
```shell
kubectl logs hazelcast -n hazelcast
```
Output:
```shell
########################################
# JAVA=/usr/bin/java
# JAVA_OPTS=--add-modules java.se --add-exports java.base/jdk.internal.ref=ALL-UNNAMED --add-opens java.base/java.lang=ALL-UNNAMED --add-opens java.base/sun.nio.ch=ALL-UNNAMED --add-opens java.management/sun.management=ALL-UNNAMED --add-opens jdk.management/com.sun.management.internal=ALL-UNNAMED -Dhazelcast.logging.type=log4j2 -Dlog4j.configurationFile=file:/opt/hazelcast/config/log4j2.properties -Dhazelcast.config=/opt/hazelcast/config/hazelcast-docker.xml -Djet.custom.lib.dir=/opt/hazelcast/custom-lib -Djava.net.preferIPv4Stack=true -XX:MaxRAMPercentage=80.0 -XX:MaxGCPauseMillis=5
# CLASSPATH=/opt/hazelcast/*:/opt/hazelcast/lib:/opt/hazelcast/lib/*:/opt/hazelcast/bin/user-lib:/opt/hazelcast/bin/user-lib/*
########################################
2023-02-23 01:21:04,236 [ INFO] [main] [c.h.i.c.AbstractConfigLocator]: Loading configuration '/opt/hazelcast/config/hazelcast-docker.xml' from System property 'hazelcast.config'
2023-02-23 01:21:04,237 [ INFO] [main] [c.h.i.c.AbstractConfigLocator]: Using configuration file at /opt/hazelcast/config/hazelcast-docker.xml
2023-02-23 01:21:04,855 [ INFO] [main] [c.h.s.logo]: [10.0.2.91]:5701 [dev] [5.2.2] 
	+       +  o    o     o     o---o o----o o      o---o     o     o----o o--o--o
	+ +   + +  |    |    / \       /  |      |     /         / \    |         |   
	+ + + + +  o----o   o   o     o   o----o |    o         o   o   o----o    |   
	+ +   + +  |    |  /     \   /    |      |     \       /     \       |    |   
	+       +  o    o o       o o---o o----o o----o o---o o       o o----o    o   
2023-02-23 01:21:04,855 [ INFO] [main] [c.h.system]: [10.0.2.91]:5701 [dev] [5.2.2] Copyright (c) 2008-2022, Hazelcast, Inc. All Rights Reserved.
2023-02-23 01:21:04,855 [ INFO] [main] [c.h.system]: [10.0.2.91]:5701 [dev] [5.2.2] Hazelcast Platform 5.2.2 (20230215 - a221adc) starting at [10.0.2.91]:5701
2023-02-23 01:21:04,855 [ INFO] [main] [c.h.system]: [10.0.2.91]:5701 [dev] [5.2.2] Cluster name: dev
2023-02-23 01:21:04,855 [ INFO] [main] [c.h.system]: [10.0.2.91]:5701 [dev] [5.2.2] Integrity Checker is disabled. Fail-fast on corrupted executables will not be performed. For more information, see the documentation for Integrity Checker.
2023-02-23 01:21:04,855 [ INFO] [main] [c.h.system]: [10.0.2.91]:5701 [dev] [5.2.2] Jet is enabled
2023-02-23 01:21:05,317 [ INFO] [main] [c.h.s.d.i.DiscoveryService]: [10.0.2.91]:5701 [dev] [5.2.2] Auto-detection selected discovery strategy: class com.hazelcast.kubernetes.HazelcastKubernetesDiscoveryStrategyFactory
2023-02-23 01:21:05,321 [ INFO] [main] [c.h.s.d.i.DiscoveryService]: [10.0.2.91]:5701 [dev] [5.2.2] Kubernetes Discovery properties: { service-dns: null, service-dns-timeout: 5, service-name: null, service-port: 0, service-label: null, service-label-value: true, namespace: hazelcast, pod-label: null, pod-label-value: null, resolve-not-ready-addresses: true, expose-externally-mode: AUTO, use-node-name-as-external-address: false, service-per-pod-label: null, service-per-pod-label-value: null, kubernetes-api-retries: 3, kubernetes-master: https://kubernetes.default.svc}
2023-02-23 01:21:05,651 [ INFO] [main] [c.h.s.d.i.DiscoveryService]: [10.0.2.91]:5701 [dev] [5.2.2] Kubernetes Discovery activated with mode: KUBERNETES_API
2023-02-23 01:21:05,652 [ INFO] [main] [c.h.s.security]: [10.0.2.91]:5701 [dev] [5.2.2] Enable DEBUG/FINE log level for log category com.hazelcast.system.security  or use -Dhazelcast.security.recommendations system property to see 🔒 security recommendations and the status of current config.
2023-02-23 01:21:05,731 [ INFO] [main] [c.h.i.i.Node]: [10.0.2.91]:5701 [dev] [5.2.2] Using Discovery SPI
2023-02-23 01:21:05,736 [ WARN] [main] [c.h.c.CPSubsystem]: [10.0.2.91]:5701 [dev] [5.2.2] CP Subsystem is not enabled. CP data structures will operate in UNSAFE mode! Please note that UNSAFE mode will not provide strong consistency guarantees.
2023-02-23 01:21:05,988 [ INFO] [main] [c.h.j.i.JetServiceBackend]: [10.0.2.91]:5701 [dev] [5.2.2] Setting number of cooperative threads and default parallelism to 2
2023-02-23 01:21:06,049 [ INFO] [main] [c.h.i.d.Diagnostics]: [10.0.2.91]:5701 [dev] [5.2.2] Diagnostics disabled. To enable add -Dhazelcast.diagnostics.enabled=true to the JVM arguments.
2023-02-23 01:21:06,053 [ INFO] [main] [c.h.c.LifecycleService]: [10.0.2.91]:5701 [dev] [5.2.2] [10.0.2.91]:5701 is STARTING
2023-02-23 01:21:06,098 [ INFO] [main] [c.h.s.d.i.DiscoveryService]: [10.0.2.91]:5701 [dev] [5.2.2] Cannot fetch the current zone, ZONE_AWARE feature is disabled
2023-02-23 01:21:06,117 [ WARN] [main] [c.h.s.d.i.DiscoveryService]: [10.0.2.91]:5701 [dev] [5.2.2] Cannot fetch name of the node, NODE_AWARE feature is disabled
2023-02-23 01:21:06,137 [ WARN] [main] [c.h.k.KubernetesClient]: Kubernetes API access is forbidden! Starting standalone. To use Hazelcast Kubernetes discovery, configure the required RBAC. For 'default' service account in 'default' namespace execute: `kubectl apply -f https://raw.githubusercontent.com/hazelcast/hazelcast/master/kubernetes-rbac.yaml`
2023-02-23 01:21:11,145 [ INFO] [main] [c.h.i.c.ClusterService]: [10.0.2.91]:5701 [dev] [5.2.2] 

Members {size:1, ver:1} [
	Member [10.0.2.91]:5701 - 98c25d23-8d5f-4413-89ff-3daec6549ed4 this
]

2023-02-23 01:21:11,172 [ INFO] [main] [c.h.j.i.JobCoordinationService]: [10.0.2.91]:5701 [dev] [5.2.2] Jet started scanning for jobs
2023-02-23 01:21:11,182 [ INFO] [main] [c.h.c.LifecycleService]: [10.0.2.91]:5701 [dev] [5.2.2] [10.0.2.91]:5701 is STARTED
```

11. see the system containers that was created for the k8s nodes:
```shell
lxc list
```
Output:
```shell
+----------+---------+--------------------------+-----------------------------------------------+-----------+-----------+
|   NAME   |  STATE  |           IPV4           |                     IPV6                      |   TYPE    | SNAPSHOTS |
+----------+---------+--------------------------+-----------------------------------------------+-----------+-----------+
| kmaster  | RUNNING | 10.85.0.1 (cni0)         | fd42:57fd:9dcd:f597:216:3eff:fe2f:bc07 (eth0) | CONTAINER | 0         |
|          |         | 10.136.18.147 (eth0)     | 1100:200::1 (cni0)                            |           |           |
|          |         | 10.0.1.184 (cilium_host) |                                               |           |           |
+----------+---------+--------------------------+-----------------------------------------------+-----------+-----------+
| kworker1 | RUNNING | 10.85.0.1 (cni0)         | fd42:57fd:9dcd:f597:216:3eff:fe59:f65d (eth0) | CONTAINER | 0         |
|          |         | 10.136.18.143 (eth0)     | 1100:200::1 (cni0)                            |           |           |
|          |         | 10.0.2.28 (cilium_host)  |                                               |           |           |
+----------+---------+--------------------------+-----------------------------------------------+-----------+-----------+
| kworker2 | RUNNING | 10.136.18.100 (eth0)     | fd42:57fd:9dcd:f597:216:3eff:fec8:7c0d (eth0) | CONTAINER | 0         |
|          |         | 10.0.0.26 (cilium_host)  |                                               |           |           |
+----------+---------+--------------------------+-----------------------------------------------+-----------+-----------+
| kworker3 | RUNNING | 10.136.18.128 (eth0)     | fd42:57fd:9dcd:f597:216:3eff:fe25:190a (eth0) | CONTAINER | 0         |
|          |         | 10.0.3.234 (cilium_host) |                                               |           |           |
+----------+---------+--------------------------+-----------------------------------------------+-----------+-----------+
```

12. Tear down the cluster now:
```shell
ansible-playbook destroy-k8s-cluster.yaml
```

13. Make sure that api server is not accessible anymore and the containers deleted:
```shell
lxc list
+------+-------+------+------+------+-----------+
| NAME | STATE | IPV4 | IPV6 | TYPE | SNAPSHOTS |
+------+-------+------+------+------+-----------+
kubectl get pods 
Unable to connect to the server: dial tcp 10.136.18.147:6443: connect: no route to host
```

14. exit from ssh session, and stop Ubuntu VM
```shell
exit
virsh --connect qemu:///system shutdown ubuntu-demo
```
