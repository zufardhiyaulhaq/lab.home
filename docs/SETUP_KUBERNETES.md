# Kubernetes Raspberry Pi setup

## Setup Postgresql storage for Kubernetes
1. add usb storage
```
lsblk
```

2. create partition for usb storage
```
sudo fdisk /dev/sda

Welcome to fdisk (util-linux 2.34).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 
First sector (2048-30031871, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-30031871, default 30031871): 

Created a new partition 1 of type 'Linux' and of size 14.3 GiB.
Partition #1 contains a ntfs signature.

Do you want to remove the signature? [Y]es/[N]o: y

The signature will be removed by a write command.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

3. format usb partition
```
sudo mkfs.ext4 -L USB_DRIVE_01 /dev/sda1
```

4. create directory for partition to mount
```
sudo mkdir /var/lib/postgresql
```

5. mount and make it permanent
```
sudo vi /etc/fstab
LABEL=USB_DRIVE_01  /var/lib/postgresql               ext4    defaults,noatime  0       1

sudo mount -a
```

6. check and make sure it's mounted
```
lsblk
```

7. install postgresql
```
sudo apt install postgresql postgresql-contrib
```

8. create database & user for kubernetes
```
sudo -u postgres psql

create database kubernetes;
create user kubernetes_user with encrypted password 'kubernetes_password';
grant all privileges on database kubernetes to kubernetes_user;
```

## Prerequisites
1. Update & Upgrade package
```
cd ansible
ansible-playbook kubernetes_update-os.yaml -i hosts/kubernetes
ansible-playbook kubernetes_enable-groups.yaml -i hosts/kubernetes
```

## Install Kubernetes master
Run k3s script to bootstrap Kubernetes master
```
sudo su
curl -sfL https://get.k3s.io | sh -s - --write-kubeconfig-mode 644 --node-taint CriticalAddonsOnly=true:NoExecute --bind-address 192.168.100.10 --disable-cloud-controller --disable local-storage --disable traefik --disable servicelb --datastore-endpoint postgres://kubernetes_user:kubernetes_password@127.0.0.1:5432/kubernetes
```

## Install Kubernetes worker
Run k3s script to bootstrap Kubernetes master
```
cd ansible
ansible-playbook kubernetes_k3s-worker-join.yaml -i hosts/kubernetes --extra-vars "kubernetes_master_url=https://192.168.100.10:6443” --extra-vars "kubernetes_token=token”
```

label Kubernetes worker
```
kubectl label nodes kubernetes-01-worker-01 kubernetes.io/role=worker
kubectl label nodes kubernetes-01-worker-02 kubernetes.io/role=worker
kubectl label nodes kubernetes-01-worker-03 kubernetes.io/role=worker
kubectl label nodes kubernetes-01-worker-04 kubernetes.io/role=worker
```

