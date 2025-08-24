## Overview

This project demonstrates the setup and hardening of a **single-node RKE2 cluster** on Ubuntu.  
The tasks included:

-  **Cluster Setup:** Installed and configured RKE2 with kubectl access.  
-  **System Audit (Lynis):** Ran a quick audit, identified missing `fail2ban`, `libpam-tmpdir`, and lack of disk encryption.  
-  **Core Deployments (Helm):**
  - NGINX Ingress Controller  
  - OpenEBS as the default StorageClass  
  - PostgreSQL (Bitnami) bound to OpenEBS PVCs and verified with queries  
-  **Security Benchmark (kube-bench):** Ran CIS checks for RKE2.  
  - Found missing audit log settings (`--audit-log-path`, `--audit-log-maxage`).  
  - Fixed by updating `/etc/rancher/rke2/config.yaml` with audit logging arguments.  

Result: A working RKE2 cluster with ingress, dynamic storage, a running PostgreSQL database, and improved security posture after Lynis and kube-bench audits.

## 1. RKE2 Installation

### Ubuntu Instructions
```bash
# stop the software firewall
sudo systemctl disable --now ufw

# update packages and install NFS client
sudo apt update
sudo apt install nfs-common -y  
sudo apt upgrade -y

# clean up unused packages
sudo apt autoremove -y
```


### Rocky Linux Instructions

```bash
# stop the software firewall
sudo systemctl disable --now firewalld

# install required packages (NFS, cryptsetup, iSCSI)
sudo dnf install -y nfs-utils cryptsetup iscsi-initiator-utils

# enable iSCSI service (needed for Longhorn storage)
sudo systemctl enable --now iscsid.service 

# update system
sudo dnf update -y

# clean up package cache
sudo dnf clean all
```

### On the first node (rke2)
```bash
# install RKE2 server
curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE=server sh -

# set bootstrap token
sudo mkdir -p /etc/rancher/rke2/ 
echo "token: bootstrapAllTheThings" | sudo tee /etc/rancher/rke2/config.yaml

# start and enable RKE2 server
sudo systemctl enable --now rke2-server.service
```
![Alt text for screen readers](https://github.com/AbdallahRSS/Bitamer_Challeng/blob/898fcd09f4453b70ef8200b632b17bcb57a8e7e0/challenge-2-platform-setup/screenshots/Screenshot%202025-08-24%20053915.png)


### Configure kubectl
```bash
# symlink kubectl
sudo ln -s $(find /var/lib/rancher/rke2/data/ -name kubectl) /usr/local/bin/kubectl

# add KUBECONFIG to bashrc for persistence
echo "export KUBECONFIG=/etc/rancher/rke2/rke2.yaml PATH=$PATH:/usr/local/bin/:/var/lib/rancher/rke2/bin/" >> ~/.bashrc
source ~/.bashrc

# verify cluster status
kubectl get node

```

![](https://github.com/AbdallahRSS/Bitamer_Challeng/blob/1ac650eb3a8290be7eaad7190cae4bb128c5db7d/challenge-2-platform-setup/screenshots/Screenshot%202025-08-24%20054032.png)

## 2. Run Lynis for a security audit

```bash
sudo apt update && sudo apt install -y lynis
sudo lynis audit system --quick
```
![](https://github.com/AbdallahRSS/Bitamer_Challeng/blob/edb7458080c7309a932adc8d6952eb253cc283c2/challenge-2-platform-setup/screenshots/Screenshot%202025-08-24%20055033.png)


![](https://github.com/AbdallahRSS/Bitamer_Challeng/blob/9688768861a26f632b97c49211afec6ad18d2dc4/challenge-2-platform-setup/screenshots/Screenshot%202025-08-24%20055358.png)

### Key Findings

Warning (DEB-0001): One package test took too long → not critical, but indicates slow execution during the audit.

libpam-tmpdir:  Not installed

Risk: temporary files in /tmp may be shared between users.

Fix: install libpam-tmpdir to isolate user tmp directories.

Disk Encryption:  Not enabled on /, /boot, and snap partitions.

Risk: if the node is compromised physically, data could be read.

Fix: use full-disk encryption (LUKS/dm-crypt) if this is a production system.

Software Checks:

apt-listbugs:  Not installed (warns of severe bugs before updates).

apt-listchanges:  Not installed (shows changelogs before updates).

needrestart:  Installed (ensures services are restarted after upgrades).

fail2ban:  Not installed (important to protect SSH against brute force).

## Install via Helm : NGINX Ingress Controller, OpenEBS, and PostgreSQL.

### Install Helm

```bash
# on the server rancher-01
# add helm
curl -#L https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# add needed helm charts
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest --force-update
helm repo add jetstack https://charts.jetstack.io --force-update
```

![](https://github.com/AbdallahRSS/Bitamer_Challeng/blob/ced801ad6bcd790e9ea9bbda4610c8a8740e787e/challenge-2-platform-setup/screenshots/Screenshot%202025-08-24%20054239.png)

### Install NGINX Ingress Controller
RKE2 comes with **NGINX Ingress Controller pre-installed** in the `kube-system` namespace.  
This means we do not need to install a separate ingress-nginx via Helm.

Verification:
```bash
kubectl get pods -n kube-system | grep ingress
```



![](https://github.com/AbdallahRSS/Bitamer_Challeng/blob/4dc367cac4c0dd65edaf16e54cf7642121d24186/challenge-2-platform-setup/screenshots/Screenshot%202025-08-24%20060217.png)

### Install OpenEBS

```bash
helm repo add openebs https://openebs.github.io/charts
helm repo update
helm install openebs openebs/openebs -n openebs --create-namespace

kubectl get sc


```
![](https://github.com/AbdallahRSS/Bitamer_Challeng/blob/d515c70e6f4ced7c23cec6a2ed6b75d6e9f3ad60/challenge-2-platform-setup/screenshots/Screenshot%202025-08-24%20060421.png)

Make openebs-hostpath the default StorageClass.

```bash
kubectl annotate sc openebs-hostpath storageclass.kubernetes.io/is-default-class="true" --overwrite
```


![]()
![]()
![]()
![]()
![]()
![]()





