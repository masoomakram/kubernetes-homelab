📄 KubeRise OpsLab — Setup Documentation (Phase 1)

🧱 Project: KubeRise OpsLab

Goal: Build a Kubernetes cluster using kubeadm on Rocky Linux VMs while learning system internals and networking.

🖥️ Infrastructure
Hypervisor: VMware Workstation Pro
OS: Rocky Linux 9
Nodes
kr-control-01 (Control Plane)
kr-worker-01
kr-worker-02

🌐 Network Configuration
Static IPs
Node	IP Address
kr-control-01	192.168.14.128
kr-worker-01	192.168.14.130
kr-worker-02	192.168.14.131

Gateway: 192.168.14.2

Static IP Setup (nmcli)
nmcli con mod ens160 ipv4.addresses <IP>/24
nmcli con mod ens160 ipv4.gateway 192.168.14.2
nmcli con mod ens160 ipv4.dns "8.8.8.8 1.1.1.1"
nmcli con mod ens160 ipv4.method manual
nmcli con up ens160
/etc/hosts (All Nodes)
192.168.14.128 kr-control-01
192.168.14.130 kr-worker-01
192.168.14.131 kr-worker-02

🔍 Connectivity Validation
ping -c 3 kr-control-01
ping -c 3 kr-worker-01

🔐 Firewall Configuration

Using firewalld with required Kubernetes ports instead of disabling it.

Control Plane
firewall-cmd --permanent --add-port=6443/tcp
firewall-cmd --permanent --add-port=2379-2380/tcp
firewall-cmd --permanent --add-port=10250/tcp
firewall-cmd --permanent --add-port=10257/tcp
firewall-cmd --permanent --add-port=10259/tcp
firewall-cmd --reload
Workers
firewall-cmd --permanent --add-port=10250/tcp
firewall-cmd --permanent --add-port=30000-32767/tcp
firewall-cmd --reload

📦 Container Runtime Setup
Install containerd
dnf install -y containerd.io
Fix Default Config (CRI Disabled Issue)

The default config had:

disabled_plugins = ["cri"]

This prevents Kubernetes from communicating with containerd.

Generate Kubernetes-Compatible Config
mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml
Enable systemd cgroup
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
Start and Enable Service
systemctl enable --now containerd
systemctl restart containerd
Verification
crictl info | grep -i cgroup

Expected:

SystemdCgroup: true

☸️ Kubernetes Tools Installed

Installed on all nodes:

dnf install -y kubeadm kubelet kubectl
Enable kubelet
systemctl enable --now kubelet

🧠 Key Learnings So Far
Static IP configuration using nmcli
Difference between:
listening ports
open ports
reachable ports
Firewall vs service-level access
CRI importance in Kubernetes
Container runtime configuration pitfalls
Basic network troubleshooting:
ping
nc
nmap
ss
Understanding systemd cgroup alignment

🚧 Current Status
All nodes configured
Networking working
Container runtime ready
Kubernetes tools installed

Cluster initialization pending.

🚀 Next Steps
kubeadm init on control plane
Configure kubectl
Join worker nodes
Install CNI (Calico/Flannel)
