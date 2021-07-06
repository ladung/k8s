# Sử dụng PersistentVolume NFS trong K8s
##Cài đặt NFS server
- Cài đặt nfs-server trên máy master
*Tham khảo: https://www.tecmint.com/install-nfs-server-on-ubuntu/

##Tạo PersistentVolume NFS
*tạo file pv-nfs.yaml
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1
spec:
  storageClassName: mystorageclass
  capacity:
    storage: 4Gi
  accessModes:
    - ReadWriteMany
  nfs:
    path: "/data/mydata/"
    server: "192.168.88.130"
```

**triển khai và kiểm trra
```
kubectl apply -f 1-pv-nfs.yaml
kubectl get pv -o wide
kubectl describe pv/pv1
```

##Tạo PersistentVolumeClaim NFS
*tạo file pvc-nfs.yaml
```
apiVersion: v1
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc1
spec:
  storageClassName: mystorageclass
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```
**triển khai và kiểm trra
```
kubectl apply -f 2-pvc-nfs.yaml
kubectl get pvc,pv -o wide
```

##Mount PersistentVolumeClaim NFS vào Container
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd
  labels:
    app: httpd
spec:
  replicas: 1
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
    spec:
      volumes:
        - name: htdocs
          persistentVolumeClaim:
            claimName: pvc1
      containers:
      - name: app
        image: httpd
        resources:
          limits:
            memory: "100M"
            cpu: "100m"
        ports:
          - containerPort: 80
        volumeMounts:
          - mountPath: /usr/local/apache2/htdocs/
            name: htdocs
---
apiVersion: v1
kind: Service
metadata:
  name: httpd
  labels:
    run: httpd
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
    nodePort: 31080
  selector:
    app: httpd
```

#Tham khảo:
- https://xuanthulab.net/su-dung-persistentvolume-nfs-tren-kubernetes.html
- https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-volumes-example-nfs-persistent-volume.html
