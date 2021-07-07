# Replicaset
- Replicaset là loại resource dùng để tạo và duy trì số lượng Pod được chỉ định. Chẳng hạn như muốn tạo 5 Pod và muốn luôn duy trì số lượng này (nhằm đảm bảo ứng dụng hoạt động ổn định). Trong trường hợp vì lý do nào đó một hoặc một vài Pod bị sự cố, k8s sẽ tự động tái tạo lại đủ 5 pod như mong muốn.

## Tạo một Replicaset
- Tạo file ```rs_sample.yml```
```
#rs_sample.yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: sample-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.12
          ports:
            - containerPort: 80
```
- Triển khai:
```
$ kubectl apply -f ./rs_sample.yml
replicaset.apps/sample-rs created
#Bây giờ chúng ta kiểm ra ReplicaSet đã được tạo Ok chưa, có tạo 3 Pod cho chúng ta không.

$ kubectl get rs -o wide
NAME        DESIRED   CURRENT   READY     AGE       CONTAINERS        IMAGES       SELECTOR
sample-rs   3         3         3         14m       nginx-container   nginx:1.12   app=sample-app
#Tiếp theo thì kiếm tra số Pod được tạo ra, trạng thái như thế nào.

$ kubectl get pod -l app=sample-app -o wide
NAME              READY     STATUS    RESTARTS   AGE       IP          NODE
sample-rs-pdkcl   1/1       Running   0          16m       10.42.2.6   Node1
sample-rs-vwz57   1/1       Running   0          16m       10.42.1.6   Node2
sample-rs-znzkg   1/1       Running   0          16m       10.42.2.5   Node1
```

## Auto Healing
- Trường hợp Node hay Pod gặp sự cố, Replicaset có cơ chế tự động khởi động lại Pod trên một Node khác, giảm sự ảnh hưởng đến hệ thống. Một trong số những khái niệm cự kỳ quan trọng trong Kubernetes đó là "Auto Healing - tự động chữa bệnh".
- Để kiểm chứng cơ chế auto healing thì ta thử xoá 1 Pod đi xem sao. Hiện tại chúng ta có 3 Pod như dưới đây, mọi người để ý NAME của nó nhé.

```
$ kubectl get pod -l app=sample-app -o wide
NAME              READY     STATUS    RESTARTS   AGE       IP          NODE
sample-rs-pdkcl   1/1       Running   0          47m       10.42.2.6   Node1
sample-rs-vwz57   1/1       Running   0          47m       10.42.1.6   Node2
sample-rs-znzkg   1/1       Running   0          47m       10.42.2.5   Node1
#Xoá Pod

$ kubectl delete pods sample-rs-pdkcl
pod "sample-rs-pdkcl" deleted
#Đã xoá xong rồi, nhưng bây giờ kiểm tra lại số lượng Pod chúng ta đang có là bao nhiêu.

$ kubectl get pod -l app=sample-app -o wide
NAME              READY     STATUS    RESTARTS   AGE       IP          NODE
sample-rs-4bwhw   1/1       Running   0          47m       10.42.2.6   Node1
sample-rs-vwz57   1/1       Running   0          47m       10.42.1.6   Node2
sample-rs-znzkg   1/1       Running   0          47m       10.42.2.5   Node1
```

## Label
- ReplicaSet quản lý các Pod được tạo ra bằng cơ chế gắn nhãn cho từng Pod. Vì vậy chúng ta dễ dàng tìm kiếm những Pod có label giống nhau, dễ dàng cho việc điều chỉnh tăng giảm số lượng Pod.
Label của Pod được chỉ định ở phần spec.selector
```
spec:
  replicas: 3
  selector:
    matchLabels:
      app: sample-app
```
*Nếu tạo 1 Pod riêng (không phải tạo trong RepicaSet Manifest) nhưng được gắn label cũng là "app:sample-app" thì điều gì xãy ra. Trong trường hợp lable bị duplicate, tức là chúng ta đang chỉ định tạo 3 Pod có label là "app:sample-app" nhưng tự nhiên ở đâu xuất hiện 1 Pod nữa cũng có nhãn như vậy. Lúc này ReplicaSet hiểu lầm là số lượng Pod đã tăng quá mức quy định, nó sẽ stop đi một Pod.
Test thử thôi!
```
#pod_sample.yml
apiVersion: v1
kind: Pod
metadata:
  name: sample-pod
  labels:
    app: sample-app
spec:
  containers:
    - name: nginx-container
      image: nginx:1.13
      ports:
      - containerPort: 80
```
```
# create Pod
$ kubectl apply -f pod_sample.yml
pod/sample-pod created
Kiểm tra tình trạng các Pod như thế nào

$ kubectl get pods -L app
NAME              READY     STATUS        RESTARTS   AGE       APP
sample-pod        0/1       Terminating   0          3s        sample-app
sample-rs-4bwhw   1/1       Running       0          2h        sample-app
sample-rs-vwz57   1/1       Running       0          3h        sample-app
sample-rs-znzkg   1/1       Running       0          3h        sample-app
Vài giây sau chúng ta kiểm tra lại lần nữa.

$ kubectl get pods -L app
NAME              READY     STATUS    RESTARTS   AGE       APP
sample-rs-4bwhw   1/1       Running   0          2h        sample-app
sample-rs-vwz57   1/1       Running   0          3h        sample-app
sample-rs-znzkg   1/1       Running   0          3h        sample-app
```
- Pod chúng ta vừa tạo đã biến mất. Vậy thì cơ chế luôn giữ số lượng Pod được chỉ định đã xoá đi 1 Pod thừa. Trong trường hợp lần này là xoá đi Pod chúng ta vừa thêm vào, nhưng cũng có trường hợp nó xoá đi Pod đã tồn tại trước đó. Vì vậy chúng ta cần lưu ý việc đánh nhãn cho Pod.
- Khi chúng ta quen với cách làm việc của Kubernetes thì Lable nên là duy nhất. Vì chúng ta có thể gắn nhiều lable cho cùng 1 container nên hãy quyết định quy tắc đặt lên lable như gợi ý sau:
```
labels:
  env: dev
  codename: system_a
  role: web-front
```

## Scaling Pod
- Bây giờ chúng ta sẽ thực hiện thay đổi số lượng Pod bằng cách thay đổi setting của ReplicaSet.
*Có 2 phương pháp:*
  - Edit file yaml config, chạy command "kubectl apply -f [FILENAME]"
  - Thực hiện scale bằng cách chạy command kubectl scale
- Cách thứ nhất thay đổi file YAML config, chạy command kubectl apply ... là sẽ update được resource. Lần này mình sẽ demo bằng kubectl scale. Nhưng Workload resource có thể thực hiện scale bằng kubectl scale chỉ có 4 loại là ```Deployment, Job, ReplicaSet, ReplicationController.```
```
$ kubectl get rs
NAME        DESIRED   CURRENT   READY     AGE
sample-rs   3         3         3         3h
$ kubectl scale rs sample-rs --replicas 5
replicaset.extensions/sample-rs scaled
$ kubectl get rs
NAME        DESIRED   CURRENT   READY     AGE
sample-rs   5         5         5         3h
$ kubectl get pods -l app=sample-app
NAME              READY     STATUS    RESTARTS   AGE
sample-rs-4bwhw   1/1       Running   0          3h
sample-rs-lz6x7   1/1       Running   0          44s
sample-rs-vwz57   1/1       Running   0          3h
sample-rs-z5l8b   1/1       Running   0          44s
sample-rs-znzkg   1/1       Running   0          3h
```

# Tham khảo:
- https://blog.vietnamlab.vn/nhap-mon-kubernetes-p4-kubernetes-workloads-resource-1/