# Kubernetes Cluster Kurulum Kılavuzu

## İçindekiler
1. [Hostname Ayarları](#1-hostname-ayarları-tüm-nodelar)
2. [Hosts Dosyası Ayarları](#2-hosts-dosyası-ayarları-tüm-nodelar)
3. [Ön Gereksinimler](#3-ön-gereksinimler-tüm-nodelar)
4. [Containerd Kurulumu](#4-containerd-kurulumu-tüm-nodelar)
5. [Kubernetes Paketleri](#5-kubernetes-paketleri-kurulumu-tüm-nodelar)
6. [Cluster Başlatma](#6-cluster-başlatma-sadece-master-node)
7. [CNI Kurulumu](#7-cni-calico-kurulumu-sadece-master-node)
8. [Worker Node'ları Ekleme](#8-worker-nodeları-clustera-ekleme-sadece-worker-nodelar)
9. [Durum Kontrolü](#9-node-durumunu-kontrol-etme-sadece-master-node)

## 1. Hostname Ayarları (Tüm Node'lar)

Master ve Worker node'ların hostnamelerini belirleyerek tanımlama yapılır. Bu sayede node'lar birbirlerini isimleriyle tanıyabilir.

```bash
# Master Node'da (192.168.200.153)
sudo hostnamectl set-hostname k8s-master

# Worker1 Node'da (192.168.200.151)
sudo hostnamectl set-hostname k8s-worker1

# Worker2 Node'da (192.168.200.152)
sudo hostnamectl set-hostname k8s-worker2
```

## 2. Hosts Dosyası Ayarları (Tüm Node'lar)

Node'ların IP adreslerini ve hostname'lerini eşleştirme yapılır. Bu sayede node'lar birbirleriyle hostname üzerinden iletişim kurabilir.

```bash
sudo cat << EOF >> /etc/hosts
192.168.200.153 k8s-master
192.168.200.151 k8s-worker1
192.168.200.152 k8s-worker2
EOF
```

## 3. Ön Gereksinimler (Tüm Node'lar)

### SELinux Ayarları
Kubernetes'in container'lar ile düzgün çalışabilmesi için SELinux permissive moda alınır.

```bash
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

### Swap Devre Dışı Bırakma
Kubernetes'in performans ve kararlılığı için swap alanı kapatılır.

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

### Kernel Modülleri
Container runtime ve network bridge için gerekli kernel modülleri yüklenir.

```bash
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

### Sistem Ayarları
Kubernetes networking için gerekli sistem ayarları yapılır.

```bash
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system
```

## 4. Containerd Kurulumu (Tüm Node'lar)

### Repository Ekleme
```bash
sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
```

### Kurulum ve Yapılandırma
```bash
# Containerd kurulumu
sudo dnf install -y containerd.io

# Yapılandırma
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```

## 5. Kubernetes Paketleri Kurulumu (Tüm Node'lar)

### Repository Ekleme
```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.28/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.28/rpm/repodata/repomd.xml.key
EOF
```

### Kubernetes Bileşenleri Kurulumu
```bash
sudo dnf install -y kubelet kubeadm kubectl
sudo systemctl enable --now kubelet
```

## 6. Cluster Başlatma (Sadece Master Node)

### Kubernetes Cluster'ı Başlatma
```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.200.153
```

### kubectl Yapılandırması
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## 7. CNI (Calico) Kurulumu (Sadece Master Node)

Pod'lar arası network iletişimi için Calico network plugin'i kurulur.

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml
```

## 8. Worker Node'ları Cluster'a Ekleme (Sadece Worker Node'lar)

Master node'dan alınan join komutu ile worker node'lar cluster'a eklenir.

```bash
sudo kubeadm join 192.168.200.153:6443 --token ie9rsv.jxo691a7upcm1pm9 \
        --discovery-token-ca-cert-hash sha256:84d34ec05c0facd6ab7dae4dddeed8ec0974bafc0851cf1a7f384c90cea97436
```

## 9. Node Durumunu Kontrol Etme (Sadece Master Node)

Cluster'daki node'ları listeleme ve durumlarını kontrol etme:

```bash
kubectl get nodes
```

---
*Not: Bu dokümantasyon, Kubernetes 1.28 sürümü için hazırlanmıştır.*