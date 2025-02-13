# Kubernetes Temel Komutlar Referansı

## Cluster Yönetimi Komutları

### Node İşlemleri
```bash
# Node'ları listele
kubectl get nodes
kubectl get nodes -o wide  # Detaylı liste

# Node detaylarını görüntüle
kubectl describe node <node-adı>

# Node'u bakım moduna al
kubectl drain <node-adı>

# Node'u cluster'dan çıkar
kubectl delete node <node-adı>
```

### Namespace İşlemleri
```bash
# Namespace'leri listele
kubectl get namespaces

# Namespace oluştur
kubectl create namespace <namespace-adı>

# Namespace sil
kubectl delete namespace <namespace-adı>
```

## Pod ve Deployment İşlemleri

### Pod Komutları
```bash
# Pod'ları listele
kubectl get pods
kubectl get pods -A  # Tüm namespace'lerdeki pod'lar
kubectl get pods -n <namespace>  # Belirli namespace'deki pod'lar
kubectl get pods -o wide  # Detaylı pod listesi

# Pod detaylarını görüntüle
kubectl describe pod <pod-adı>

# Pod loglarını görüntüle
kubectl logs <pod-adı>
kubectl logs -f <pod-adı>  # Canlı log takibi

# Pod'a bağlan
kubectl exec -it <pod-adı> -- /bin/bash

# Pod'u sil
kubectl delete pod <pod-adı>
```

### Deployment İşlemleri
```bash
# Deployment oluştur
kubectl create deployment <deployment-adı> --image=<imaj-adı>

# Deployment'ları listele
kubectl get deployments

# Deployment ölçeklendir
kubectl scale deployment <deployment-adı> --replicas=<sayı>

# Deployment güncelle
kubectl set image deployment/<deployment-adı> <container-adı>=<yeni-imaj>

# Deployment silme
kubectl delete deployment <deployment-adı>
```

## Service ve Network İşlemleri

### Service Komutları
```bash
# Service oluştur
kubectl expose deployment <deployment-adı> --port=<port> --type=NodePort

# Service'leri listele
kubectl get services
kubectl get svc  # Kısa versiyon

# Service detaylarını görüntüle
kubectl describe service <service-adı>

# Service sil
kubectl delete service <service-adı>
```

## Kaynak İzleme ve Debugging

### İzleme Komutları
```bash
# Kaynak kullanımını görüntüle
kubectl top nodes
kubectl top pods

# Event'ları görüntüle
kubectl get events

# Pod loglarını görüntüle
kubectl logs <pod-adı> -f
```

## Konfigürasyon İşlemleri

### ConfigMap ve Secret İşlemleri
```bash
# ConfigMap oluştur
kubectl create configmap <configmap-adı> --from-file=<dosya-yolu>

# Secret oluştur
kubectl create secret generic <secret-adı> --from-literal=username=admin

# ConfigMap ve Secret'ları listele
kubectl get configmaps
kubectl get secrets
```

## YAML İşlemleri

### YAML Dosyaları ile Çalışma
```bash
# YAML dosyasından oluştur
kubectl apply -f <dosya.yaml>

# YAML dosyasından sil
kubectl delete -f <dosya.yaml>

# Mevcut kaynağın YAML'ını görüntüle
kubectl get pod <pod-adı> -o yaml
```

## Context ve Cluster Yönetimi

### Context İşlemleri
```bash
# Context'leri listele
kubectl config get-contexts

# Aktif context'i değiştir
kubectl config use-context <context-adı>

# Mevcut context'i görüntüle
kubectl config current-context
```

## Label ve Annotation İşlemleri

### Label Komutları
```bash
# Label ekle
kubectl label pods <pod-adı> key=value

# Label'a göre filtrele
kubectl get pods -l key=value

# Label sil
kubectl label pods <pod-adı> key-
```

## Kullanışlı Kısayollar
```bash
# Kısaltmalar
po -> pods
svc -> services
deploy -> deployments
ns -> namespaces
pv -> persistentvolumes
pvc -> persistentvolumeclaims
```

## İpuçları

1. Watch modu için `-w` kullanın:
```bash
kubectl get pods -w
```

2. Çıktı formatını değiştirmek için `-o` kullanın:
```bash
-o wide  # Detaylı çıktı
-o yaml  # YAML formatında
-o json  # JSON formatında
```

3. Namespace seçimi için `-n` kullanın:
```bash
kubectl get pods -n kube-system
```
