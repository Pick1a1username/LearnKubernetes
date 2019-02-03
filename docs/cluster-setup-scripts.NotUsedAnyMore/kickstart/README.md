Install a single node, get it's kickstart file. Adjust file and create multiple kickstart files from it.
```
[kamran@kworkhorse ansible]$ sed -e 's/NODEIP/10.240.0.12/' -e 's/NODEFQDN/etcd2.example.com/' kickstart-template.ks  > etcd2.ks
[kamran@kworkhorse ansible]$ sed -e 's/NODEIP/10.240.0.13/' -e 's/NODEFQDN/etcd3.example.com/' kickstart-template.ks  > etcd3.ks
[kamran@kworkhorse ansible]$ sed -e 's/NODEIP/10.240.0.21/' -e 's/NODEFQDN/controller1.example.com/' kickstart-template.ks  > controller1.ks
[kamran@kworkhorse ansible]$ sed -e 's/NODEIP/10.240.0.22/' -e 's/NODEFQDN/controller2.example.com/' kickstart-template.ks  > controller2.ks
[kamran@kworkhorse ansible]$ sed -e 's/NODEIP/10.240.0.31/' -e 's/NODEFQDN/worker1.example.com/' kickstart-template.ks  > worker1.ks
[kamran@kworkhorse ansible]$ sed -e 's/NODEIP/10.240.0.32/' -e 's/NODEFQDN/worker2.example.com/' kickstart-template.ks  > worker2.ks
[kamran@kworkhorse ansible]$ sed -e 's/NODEIP/10.240.0.41/' -e 's/NODEFQDN/lb1.example.com/' kickstart-template.ks  > lb1.ks
[kamran@kworkhorse ansible]$ sed -e 's/NODEIP/10.240.0.42/' -e 's/NODEFQDN/lb2.example.com/' kickstart-template.ks  > lb2.ks

[kamran@kworkhorse kickstart-files]$ sed -e 's/NODE_IP/10.240.0.51/' -e 's/NODE_FQDN/test.example.com/' -e 's/NODE_NETMASK/255.255.255.0/' -e 's/NODE_GATEWAY/10.240.0.1/'  -e 's/NODE_DNS/10.240.0.1/'   kickstart-template.ks.working  > test.ks

```


# Serve Fedora DVD and the kickstart files, over HTTPD:

First mount the Fedora ISO on a mount point.

```
[root@kworkhorse cdimages]# mount -o loop /home/cdimages/Fedora-Server-dvd-x86_64-24-1.2.iso  /mnt/cdrom/
mount: /dev/loop2 is write-protected, mounting read-only
[root@kworkhorse cdimages]#
```

Start a docker container exposing port 80 on work computer. Serve cdrom and kickstart  directories.

```
[kamran@kworkhorse cluster-setup-scripts]$ docker run -v /mnt/cdrom:/usr/local/apache2/htdocs/cdrom   -v $(pwd)/kickstart:/usr/local/apache2/htdocs/kickstart  -p 80:80 -d httpd
bc90d48d31aca8393877af160838dd12fc44b9931d6edc22c744ebe07be3c45f


[kamran@kworkhorse cluster-setup-scripts]$ docker ps
CONTAINER ID        IMAGE               COMMAND              CREATED             STATUS              PORTS                NAMES
bc90d48d31ac        httpd               "httpd-foreground"   7 seconds ago       Up 6 seconds        0.0.0.0:80->80/tcp   condescending_brattain
[kamran@kworkhorse cluster-setup-scripts]$ 
```





# Virt-install commands:


```
  848  virt-install  -n etcd1  --description "etcd1"  --hvm  --os-type=Linux  --os-variant=fedora22  --ram=512  --vcpus=1   --disk path=/home/virtualmachines/etcd1.qcow2,bus=virtio,size=4   --location http://10.240.0.1/cdrom/  --network network=Kubernetes --extra-args "ks=http://10.240.0.1/etcd1.ks"
  869  virt-install  -n etcd1  --description "etcd1"  --hvm  --os-type=Linux  --os-variant=fedora22  --ram=512  --vcpus=1   --disk path=/home/virtualmachines/etcd1.qcow2,bus=virtio,size=4   --location http://10.240.0.1/cdrom/  --network network=Kubernetes --extra-args "ks=http://10.240.0.1/ks/etcd1.ks"
  906  history | grep virt-install
  907  virt-install -n ftest --description "test fedora24" --hvm  --os-type=Linux  --os-variant=fedora22  --ram=512  --vcpus=1   --disk path=/home/virtualmachines/ftest.qcow2,bus=virtio,size=4   --location http://10.240.0.1/fedora24/ --network network=Kubernetes 
  908  virt-install -n ftest --description "test fedora24" --hvm  --os-type=Linux  --os-variant=fedora22  --ram=512  --vcpus=1   --disk path=/home/virtualmachines/ftest.qcow2,bus=virtio,size=4   --location http://192.168.124.1/fedora24/ --network network=Kubernetes 
  909  virt-install -n ftest --description "test fedora24" --hvm  --os-type=Linux  --os-variant=fedora22  --ram=1024  --vcpus=1   --disk path=/home/virtualmachines/ftest.qcow2,bus=virtio,size=4   --location http://192.168.124.1/fedora24/ --network network=Kubernetes 
  910  virt-install -n ftest --description "test fedora24" --hvm  --os-type=Linux  --os-variant=fedora22  --ram=1024  --vcpus=1   --disk path=/home/virtualmachines/ftest.qcow2,bus=virtio,size=4   --location http://192.168.124.1/fedora24/ 
  911  virt-install -n ftest --description "test fedora24" --hvm  --os-type=Linux  --os-variant=fedora22  --ram=2048  --vcpus=1   --disk path=/home/virtualmachines/ftest.qcow2,bus=virtio,size=4   --location http://192.168.124.1/fedora24/ 


virt-install -n test --description "test fedora24" --hvm  --os-type=Linux  --os-variant=fedora22  --ram=1024  --cpu host --vcpus=1  --features acpi=on,apic=on  --clock offset=localtime  --disk path=/home/virtualmachines/test-vm.qcow2,bus=virtio,size=6  --network network=Kubernetes  --location http://10.240.0.1/ --extra-args "ks=http://10.240.0.1/kickstart/test.ks" --noautoconsole --sound=clearxml --noreboot

time virt-install -n test --description "test fedora24" --hvm  --cpu host --os-type Linux  --os-variant fedora22  --ram 1280  --vcpus 1  --features acpi=on,apic=on  --clock offset=localtime  --disk path=/home/virtualmachines/test-vm.qcow2,bus=virtio,size=6  --network network=Kubernetes  --location http://10.240.0.1/ --extra-args "ks=http://10.240.0.1/kickstart/test.ks"  --noreboot



```



Modifying a VM after installation:
```
[root@kworkhorse ~]# virt-xml lb1 --edit --memory 384,maxmemory=384
Domain 'lb1' defined successfully.
Changes will take effect after the next domain shutdown.
```



Example run:
```
[root@kworkhorse cdimages]# time virt-install -n test --description "test fedora24" --hvm  --cpu host --os-type Linux  --os-variant fedora22  --ram 1280  --vcpus 1  --features acpi=on,apic=on  --clock offset=localtime  --disk path=/home/virtualmachines/test-vm.qcow2,bus=virtio,size=6  --network network=Kubernetes  --location http://10.240.0.1/ --extra-args "ks=http://10.240.0.1/kickstart/test.ks"  --noreboot

Starting install...
Retrieving file vmlinuz...                                                                                               | 6.0 MB  00:00:00     
Retrieving file initrd.img...                                                                                            |  46 MB  00:00:00     
Creating domain...                                                                                                       |    0 B  00:00:00     

(virt-viewer:9502): GSpice-WARNING **: Warning no automount-inhibiting implementation available
Domain installation still in progress. You can reconnect to 
the console to complete the installation process.

real	4m23.375s
user	0m2.962s
sys	0m1.121s
[root@kworkhorse cdimages]# 
```


The node will shutoff after installation. At this point, change/reduce it's RAM size:

```
[root@kworkhorse cdimages]# virt-xml test --edit --memory 256,maxmemory=256
Domain 'test' defined successfully.
[root@kworkhorse cdimages]# virsh start test
Domain test started

[root@kworkhorse cdimages]# 
```

Node stats after first boot:

```
[root@test ~]# df -hT
Filesystem     Type      Size  Used Avail Use% Mounted on
devtmpfs       devtmpfs  111M     0  111M   0% /dev
tmpfs          tmpfs     119M     0  119M   0% /dev/shm
tmpfs          tmpfs     119M  532K  118M   1% /run
tmpfs          tmpfs     119M     0  119M   0% /sys/fs/cgroup
/dev/vda2      xfs       5.0G  957M  4.1G  19% /
tmpfs          tmpfs     119M     0  119M   0% /tmp
tmpfs          tmpfs      24M     0   24M   0% /run/user/0


[root@test ~]# rpm -qa | wc -l
299


[root@test ~]# getenforce 
Disabled
[root@test ~]#


[root@test ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         


[root@test ~]# service firewalld status
Redirecting to /bin/systemctl status  firewalld.service
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
[root@test ~]# 
```



