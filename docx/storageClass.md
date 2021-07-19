# StorageClass
- Mọi thứ chúng ta biết về PVs và PVC trong Kubernestes Cluster đã được trình bày trong phần trước là khá chi tiết. Tuy nhiên, nó không thể scale - ta không thể quản lý một số lượng lớn node trong Kubernetes CLuster mà tạo PVs và PVC một cách thủ công. Chúng ta cần tự động hóa -> StorageClass ra đời
- StorageClass cho phép ta định nghĩa các ```classes``` , ```tiers``` or ```storage``` khác nhau. Cách ta xác định các ```class``` là tùy ý nhưng sẽ phụ thuộc vào loại storage mà ta có quyền truy cập. Ví dụ, cần phải định nghĩa một fast class, slow class và encrypted class
- Theo Kubernetes, StorageClass được định nghĩa như resource trong storage.k8s.io/v1 API group. Resource type là StorageClass, ta định nghĩa chúng trong file YAML và POST tới API server để deploy.

Ví dụ về StorageClass:
```
kind: StorageClass
apiVersion:storage.k8s.io/v1
metadata: 
  name: fast
provisioner: kubernetes.io/aws-ebs
parameters:
  type: io1
  zones: eu-west-1a
  iopsPerGB: "10"
```
*Note*:
- ```StorageClass``` là không thể thay đổi một khi đã được deploy.
- ```metadata.name```: chỉ ra tên của class mà các đối tượng khác gọi đến.
- ```provisioner```: chỉ ra loại plugin sử dụng
- ```parameters```: chỉ ra các tham số được mô tả tương ứng với plugin sử dụng. Nó yêu cầu bạn phải có kiến thức về các plugin sử dụng.

# Multiple StorageClass
- Ta có thể cấu hình nhiều StorageClass mà ta muốn. Tuy nhiên, mỗi StorageClass chỉ được duy nhất 1 provisioner. Ví dụ, nếu ta có cụm K8s cluster với StorageOS và Portworx storage, ta cần tạo ít nhất 2 StorageClass Object.
- Mỗi loại Storage có thể cung cấp nhiều class/tiers, mỗi loại trong đó có thể có StorageClass riêng.
*Ví dụ:*
```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: portworx-db-secure
provisioner: kubernetes.io/portworx-volume
parameters:
  fs: "xfs"
  block_size: "32"
  repl: "2"
  snap_interval: "30"
  io-priority: "medium"
  secure: "true"
```

## Implementing StorageClass
- Quy trình triển khai 1 StorageClass dựa trên các bước cơ bản sau:
  1. Setup cụm k8s với Storage back-end
  2. Đảm bảo plugin storage k8s là available
  3. Create StorageClass
  4. Create PVC tham chiếu tới StorageClass bởi metadata.name
  5. Deploy POD sử dụng volume dựa trên PVC
  *Lưu ý*: Quá trình không yêu cầu tạo PV do StorageClass sẽ tự động tạo PV

**Ví dụ**: Định nghĩa file YAML với 3 object
```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: fast
# Referenced by the PVC
provisioner: kubernetes.io.gce-pd
parameters:
  type:pd-ssd
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
#Referenced by the PodSpec
  namespace: mynamespace
spec:
  accessModes:
    - ReadWriteOnce
  resources: 
  requests:
    storage: 50Gi
  storageClassName: fast
#Matches name of the SC
---
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: mypvc
#Matches PVC name
  containers:
...
<
SNIP
>
```
# Tham khảo
- https://kubernetes.io/docs/concepts/storage/storage-classes/
- The Kubernetes book - 3rd - 2019