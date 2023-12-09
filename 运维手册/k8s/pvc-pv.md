# pv and pvc

## cephfs-pvc

```yaml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: cephfs-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  cephfs:
    monitors:
      - 10.143.248.61:6789
      - 10.143.248.62:6789
      - 10.143.248.63:6789
    path: /data/replay
    user: cephfs_user
    secretRef:
      name: ceph-test-default-ceph-storage-secret
    readOnly: false
  persistentVolumeReclaimPolicy: Recycle
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: cephfs-pv-claim
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: cephfs-pvc-pod
  name: cephfs-pv-pod1
spec:
  nodeSelector:
    kubernetes.io/hostname: "kvm23-k8s-test1"
  containers:
  - name: cephfs-pv-busybox
    image: nginx:latest
    command: ["sleep", "60000"]
    volumeMounts:
    - mountPath: "/mnt/cephfs-pvc"
      name: cephfs-vol1
      readOnly: false
  volumes:
  - name: cephfs-vol1
    persistentVolumeClaim:
      claimName: cephfs-pv-claim
```

## cephfs-volume

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kube-cephfs
spec:
  containers:
  - name: cephfs-rw
    image: nginx:latest
    command: ["sleep", "60000"]
    volumeMounts:
    - mountPath: "/mnt/cephfs"
      name: cephfs
  volumes:
  - name: cephfs
    cephfs:
      monitors:
        - 10.143.248.61:6789
        - 10.143.248.62:6789
        - 10.143.248.63:6789
      user: cephfs_user
      path: /data/entry/replay
      secretRef:
        name: ceph-test-default-ceph-storage-secret
```

## local-pv

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv-zk-0
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 5Gi
  local:
    path: /data/pv/local-pv-0
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: node-role.kubernetes.io/node
          operator: In
          values:
          - ""
  persistentVolumeReclaimPolicy: Retain
  volumeMode: Filesystem
```

## block-pv

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: block-mount-pod
spec:
  containers:
    - name: myfrontend
      image: nginx:1.15
      #volumeMounts:
      #  - name: block-pv
      #   mountPath: /mnt
      volumeDevices:
        - name: block-pv
          devicePath: /dev/xvdb

  volumes:
    - name: block-pv
      persistentVolumeClaim:
        claimName: example-local-claim-block-mount
        readOnly: false
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: example-local-claim-block-mount
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  volumeMode: Block
  storageClassName: local-storage
  
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-to-mount-in-pod-on-test6
spec:
  capacity:
    storage: 20Gi
  volumeMode: Block
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  local:
    path: /dev/vdc2
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - kvm10-k8s-test6
```
