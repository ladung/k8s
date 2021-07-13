# Ingress
- Là thành phần để điều hướng các traffic HTTP và HTTPS từ bên ngoài (internet) vào các dịch vụ bên trong cluster
- Ingress chỉ để phục vụ các cổng, yêu cầu HTTP và HTTPS, còn các loại cổng khác, giao thức khác để truy cập được từ bên ngoài thì cần dùng service kiểu NodePort và Loadbalancer
- Để Ingress hoạt động, hệ thống cần một Ingress Controller

##Cài đặt ingress controller
**Tạo namespace có tên ingress-controller
```kubectl create ns ingress-controller```
**Triển khai các thành phần**
```kubectl apply -f https://haproxy-ingress.github.io/resources/haproxy-ingress.yaml```

**Gán label cho các node**
```
kubectl label node k8s-master01 role=ingress-controller
kubectl label node k8s-worker01 role=ingress-controller
kubectl label node k8s-worker02 role=ingress-controller
```

**kiếm tra các thành phần**
```kubectl get all -n ingress-controller```

##Tạo ingress
**Triển khai UD HTTP**
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: http-test-svc
  labels:
    run: http-test-svc
  namespace: haproxy-controller
spec:
  replicas: 2
  selector:
    matchLabels:
      run: http-test-app
  
  template:
    metadata:
      labels:
        run: http-test-app
    spec:
      containers:
      - name: http
        image: nginx:1.17.6
        imagePullPolicy: IfNotPresent
        resources:
          limits:
            memory: "128Mi"
            cpu: "50m"
        ports:
        - containerPort: 80
          protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: http-test-svc
  namespace: haproxy-controller
spec:
  type: ClusterIP
  selector:
    run: http-test-app
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80

```

*Triển khai
```
kubectl apply -f 1.app-test.yaml
```
- File trên triển khai một ứng dụng từ image nginx, trong đó có tạo một service với tên http-test-svc để tương tác với các POD tạo ra.
- Giờ ta sẽ tạo một Ingress để điều hướng traffic (http, https) vào dịch vụ này.
```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: app
  namespace: haproxy-controller
spec:
  rules:
  - host: ladung.test
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          serviceName: http-test-svc
          servicePort: 80
```
*triển khai*
```kubectl apply -f 2.app-test-ingress.yaml```


# Nginx ingress controller

