# ☸️ Ceph ve Kubernetes Entegrasyonu Rehberi

Bu doküman, Ceph storage'ını Kubernetes cluster'ı ile entegre etmeyi, Rook operatörü ve CSI driver kullanımını kapsar.

---

## 1. Entegrasyon Yöntemleri

### Seçenekler

| Yöntem | Açıklama | Kullanım |
|--------|----------|----------|
| **Rook-Ceph** | Kubernetes native Ceph yönetimi | Yeni kurulum |
| **External CSI** | Mevcut Ceph cluster'a bağlanma | Var olan cluster |
| **StaticPV** | Manuel PV/PVC | Test/POC |

---

## 2. Yöntem 1: Rook-Ceph Operatörü

Rook, Kubernetes içinde Ceph cluster'ı oluşturur ve yönetir.

### Rook Kurulumu

```bash
# Rook operatörünü yükle (Helm ile)
helm repo add rook-release https://charts.rook.io/release
helm install --create-namespace --namespace rook-ceph rook-ceph rook-release/rook-ceph

# Operatör pod'unu kontrol et
kubectl -n rook-ceph get pod
```

### Ceph Cluster Oluşturma

`cluster.yaml`:

```yaml
apiVersion: ceph.rook.io/v1
kind: CephCluster
metadata:
  name: rook-ceph
  namespace: rook-ceph
spec:
  cephVersion:
    image: quay.io/ceph/ceph:v18.2.1
  dataDirHostPath: /var/lib/rook
  mon:
    count: 3
    allowMultiplePerNode: false
  mgr:
    count: 2
  dashboard:
    enabled: true
  storage:
    useAllNodes: true
    useAllDevices: true
```

```bash
kubectl apply -f cluster.yaml

# Cluster durumunu kontrol et
kubectl -n rook-ceph get cephcluster
```

### Rook Toolbox

```bash
# Toolbox pod'u oluştur
kubectl -n rook-ceph apply -f https://raw.githubusercontent.com/rook/rook/release-1.13/deploy/examples/toolbox.yaml

# Ceph komutları çalıştır
kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- ceph -s
```

---

## 3. Yöntem 2: External Ceph CSI Driver

Mevcut Ceph cluster'ınızı Kubernetes'e bağlama.

### CSI Driver Kurulumu

```bash
# Ceph-CSI helm repo ekle
helm repo add ceph-csi https://ceph.github.io/csi-charts
helm repo update

# RBD CSI driver'ı kur
helm install --namespace ceph-csi --create-namespace ceph-csi-rbd ceph-csi/ceph-csi-rbd
```

### ConfigMap Oluşturma

```yaml
# csi-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: ceph-csi-config
  namespace: ceph-csi
data:
  config.json: |-
    [
      {
        "clusterID": "cluster-id-from-ceph",
        "monitors": [
          "192.168.1.10:6789",
          "192.168.1.11:6789",
          "192.168.1.12:6789"
        ]
      }
    ]
```

```bash
# Cluster ID'yi öğren (Ceph tarafında)
ceph fsid

kubectl apply -f csi-config.yaml
```

### Secret Oluşturma

```bash
# Ceph'ten keyring al
ceph auth get-or-create client.k8s mon 'profile rbd' osd 'profile rbd pool=k8s-pool' mgr 'allow rw'
```

```yaml
# csi-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: csi-rbd-secret
  namespace: ceph-csi
stringData:
  userID: k8s
  userKey: AQAJz...==
```

```bash
kubectl apply -f csi-secret.yaml
```

---

## 4. StorageClass Yapılandırması

### RBD StorageClass

```yaml
# storageclass-rbd.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ceph-rbd
provisioner: rbd.csi.ceph.com
parameters:
  clusterID: "cluster-id-from-ceph"
  pool: k8s-pool
  imageFormat: "2"
  imageFeatures: layering
  csi.storage.k8s.io/provisioner-secret-name: csi-rbd-secret
  csi.storage.k8s.io/provisioner-secret-namespace: ceph-csi
  csi.storage.k8s.io/node-stage-secret-name: csi-rbd-secret
  csi.storage.k8s.io/node-stage-secret-namespace: ceph-csi
  csi.storage.k8s.io/controller-expand-secret-name: csi-rbd-secret
  csi.storage.k8s.io/controller-expand-secret-namespace: ceph-csi
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: Immediate
```

```bash
kubectl apply -f storageclass-rbd.yaml
```

### CephFS StorageClass

```yaml
# storageclass-cephfs.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ceph-fs
provisioner: cephfs.csi.ceph.com
parameters:
  clusterID: "cluster-id-from-ceph"
  fsName: myfs
  pool: cephfs_data
  csi.storage.k8s.io/provisioner-secret-name: csi-cephfs-secret
  csi.storage.k8s.io/provisioner-secret-namespace: ceph-csi
  csi.storage.k8s.io/node-stage-secret-name: csi-cephfs-secret
  csi.storage.k8s.io/node-stage-secret-namespace: ceph-csi
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: Immediate
```

---

## 5. PVC ve Pod Kullanımı

### PVC Oluşturma

```yaml
# pvc-example.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ceph-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: ceph-rbd
  resources:
    requests:
      storage: 10Gi
```

```bash
kubectl apply -f pvc-example.yaml
kubectl get pvc
```

### Pod'da Kullanım

```yaml
# pod-example.yaml
apiVersion: v1
kind: Pod
metadata:
  name: ceph-test-pod
spec:
  containers:
    - name: app
      image: nginx
      volumeMounts:
        - name: ceph-volume
          mountPath: /data
  volumes:
    - name: ceph-volume
      persistentVolumeClaim:
        claimName: ceph-pvc
```

---

## 6. StatefulSet Örneği

```yaml
# statefulset-example.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: database
spec:
  serviceName: database
  replicas: 3
  selector:
    matchLabels:
      app: database
  template:
    metadata:
      labels:
        app: database
    spec:
      containers:
        - name: db
          image: postgres:15
          volumeMounts:
            - name: data
              mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: ceph-rbd
        resources:
          requests:
            storage: 50Gi
```

---

## 7. Volume Snapshot

### VolumeSnapshotClass Oluşturma

```yaml
# snapshotclass.yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshotClass
metadata:
  name: ceph-rbd-snapshot
driver: rbd.csi.ceph.com
deletionPolicy: Delete
parameters:
  clusterID: "cluster-id-from-ceph"
  csi.storage.k8s.io/snapshotter-secret-name: csi-rbd-secret
  csi.storage.k8s.io/snapshotter-secret-namespace: ceph-csi
```

### Snapshot Alma

```yaml
# snapshot.yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: ceph-pvc-snapshot
spec:
  volumeSnapshotClassName: ceph-rbd-snapshot
  source:
    persistentVolumeClaimName: ceph-pvc
```

### Snapshot'tan Restore

```yaml
# restore-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: restored-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: ceph-rbd
  resources:
    requests:
      storage: 10Gi
  dataSource:
    name: ceph-pvc-snapshot
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
```

---

## 8. Volume Expansion (Online Resize)

```bash
# PVC boyutunu güncelle
kubectl patch pvc ceph-pvc -p '{"spec":{"resources":{"requests":{"storage":"20Gi"}}}}'

# Durumu kontrol et
kubectl get pvc ceph-pvc
```

---

## 9. RWX (ReadWriteMany) - CephFS

Birden fazla pod'un aynı anda yazabilmesi için CephFS kullanın:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: shared-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: ceph-fs
  resources:
    requests:
      storage: 100Gi
```

---

## 10. Troubleshooting

### CSI Pod Logları

```bash
# RBD CSI nodeplugin logları
kubectl -n ceph-csi logs -l app=ceph-csi-rbd -c csi-rbdplugin

# Provisioner logları
kubectl -n ceph-csi logs -l app=ceph-csi-rbd -c csi-provisioner
```

### PVC Pending Durumu

```bash
# PVC detaylarını kontrol et
kubectl describe pvc ceph-pvc

# Yaygın sorunlar:
# 1. Secret bulunamıyor → Secret namespace/name kontrol
# 2. Pool yok → Ceph'te pool oluştur
# 3. StorageClass yanlış → ClusterID, monitors kontrol
```

### Rook-Ceph Debug

```bash
# Ceph durumu
kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- ceph -s

# OSD durumu
kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- ceph osd tree
```

---

## 11. Best Practices

### Yapılması Gerekenler

* Ayrı pool'lar kullanın (database, logs, media)
* VolumeSnapshot ile backup alın
* Resource limits tanımlayın
* StorageClass reclaim policy'yi dikkatli seçin
* CSI driver'ı güncel tutun

### Yapılmaması Gerekenler

* Admin keyring'i Kubernetes'te kullanmayın
* Tek StorageClass ile her şeyi çözmeye çalışmayın
* RWX gereken yerde RWO kullanmayın
* Snapshot olmadan production'a çıkmayın
