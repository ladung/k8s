# Sử dụng Persistent Volume (pv) và Persistent Volume Claim (pvc) trong Kubernetes
## PersistentVolume (PV)
- Là 1 resource trong k8s giúp ta truy cập vào các dich vụ lưu trữ file
***up ảnh**
- Về cơ bản PersistentVolume là một cách để lấy một từ storage của các bạn và dành phần đó cho một pod nhất định. Volume luôn được sử dụng bởi các POD chứ không phải một đối tượng cao cấp như Deployment/ReplicaSet

##ví dụ: tạo file persistent-vol.yaml như sau
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1
spec:
  capacity:
    storage: 3Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: mystorageClass
  hostPath:
    path: "/v1"
```

*Trong đó:
- ```spec.storageClassName```: cho biết ClassName của volume. Có thể sử dụng bất kỳ ClassName nào mà ta thiết lập.
- ```spec.capacity.storage```: dung lượng lưu trữ của pv
- ```spec.accessModes```: đặt chế độ truy cập cho volume. Có 3 chế độ truy cập cos thể có:
  - ```ReadWriteOnce```: volume có thể được mount dưới dạng đọc ghi bởi 1 nút duy nhất.
  - ```ReadWriteMany```: volume có thể được mount dưới dạng đọc-ghi bởi nhiều node
  - ```ReadOnlyMany```: volume có thể được mount ở chế độ chỉ đọc bởi nhiều node.
- ```hostPath```: cho biết thư mục trong trên các node để làm volume. Tham số này chỉ sử dụng khi ta tạo ra các volume mà nó dùng các thư mục của các node.
- ```persistentVolumeReclaimPolicy``` :là phần xử lý khi xảy ra trường hợp PersistentVolumeClaim tương ứng bị xóa. Các tùy chọn là Retain (giữ lại), Delete và Recycle

*Triển khai 
```
# triển khai
kubectl apply -f 1.persistent-vol.yaml

# liệt kê các PV
kubectl get pv -o wide

# thông tin chi tiết
root@k8s-master01:/home/u01# kubectl describe pv/pv1
Name:            pv1
Labels:          <none>
Annotations:     Finalizers:  [kubernetes.io/pv-protection]
StorageClass:    mystorageclass
Status:          Available
Claim:           
Reclaim Policy:  Retain
Access Modes:    RWO
VolumeMode:      Filesystem
Capacity:        3Gi
Node Affinity:   <none>
Message:         
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /v1
    HostPathType:  
Events:            <none>
```
##PersistentVolumeClaim(pvc)
- PVC là nơi yêu cầu lưu trữ cho 1 POD. Giả sử trong một cluster có nhiều volume, Claim sẽ xác định các đặc điểm mà một volume phải đáp ứng để phù hợp với nhu cầu của một POD.
- Mỗi PV thì chỉ có 1 PVC.

**Ví dụ: tạo file pvc.yaml
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc1
  labels:
    name: pvc1
spec:
  storageClassName: mystorageclass
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 150Mi
```
*Trong đó:
```spec.storageClassName```: tìm ra các pv trên cluster có để sử dụng bởi Claim. Có nghĩa là bất kỳ PersistentVolume nào có thiết lập storageClassName là ```mystorageClass``` sẽ được sử dụng bởi claim này. Nếu có nhiều volume với ```storageClassName: mystorageClass``` thì Claim sẽ nhận bất kỳ volume nào trong số đó và nếu không có volume nào với ```mystorageClass``` - một volume sẽ được cấp động.
```spec.accessModes```: thiết lập chế độ truy nhập. Có nghĩa là Claim này muốn có một lưu trữ có accessModes là ReadWriteOnce. Giả sử bạn có 2 volume với st là mystorageClass, một trong 2 volume thiết lập storageClassName thành ReadWriteOnce và cái còn lại được thiết lập là ReadWriteMany thì Claim này sẽ nhận được volume có thiết lập là ReadWriteOnce.
```resource.requests.storage```: dung lượng lưu trữ mà Claim này muốn. 150Mi không có nghĩa là nó phải có chính xác 150Mi dung lượng lưu trữ mà nó nghĩa là nó phải có ít nhất 150Mi.
* triển khai
```kubectl apply -f 2.persistent-vol-claim.yaml

kubectl get pvc,pv -o wide
kubectl describe pvc/pvc1
```
***chú ý: Nếu xóa PVC thì PV sẽ không sủ dụng được nữa, nếu muốn sử dụng ta cần thực hiện như sau:
```kubectl edit pv pv1
#Xóa đi mục ClaimRef:
#lưu lại và triển khai lại pvc1 là oke!
```

**Sử dụng PVC với Pod
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      volumes:
        - name: myvolume
          persistentVolumeClaim:
            claimName: pvc1
      containers:
      - name: myapp
        image: nginx:1.17.6
        resources:
           limits:
             memory: "50Mi"
             cpu: "100m"
        command:
          - sleep
          - "600"
        volumeMounts:
          - mountPath: "/data"
            name: myvolume
```
#Tham khảo:
- https://comdy.vn/kubernetes/so-tay-kubernetes-trien-khai-ung-dung-da-container-p2/
- https://xuanthulab.net/su-dung-persistent-volume-pv-va-persistent-volume-claim-pvc-trong-kubernetes.html
- https://kubernetes.io/docs/concepts/storage/persistent-volumes/

