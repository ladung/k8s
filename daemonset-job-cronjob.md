#DaemonSet, Job, CronJob
- DaemonSet ương tự như ReplicaSet , có khả năng tạo và quản lý các POD
- Điểm khác biệt là nó tạo ra trên mỗi node chỉ 1 POD.
- Triển khai DaemonSet khi cần ở mỗi node 1 POD, thường dùng cho các ứng dụng như thu thập log,...

##Ví dụ chạy DaemonSet với nginx:
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: dsapp
spec:
  selector:
    matchLabels:
      app: ds-nginx
  template:
    metadata:
      labels:
        app: ds-nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        resources:
          limits:
            memory: "128Mi"
            cpu: "100m"
        ports:
        - containerPort: 80
```

**Triển khai

```kubectl apply -f 1.ds.yaml

# Liệt kê các DaemonSet
kubectl get ds -o wide

# Liệt kê các POD theo nhãn
kubectl get pod -o wide -l "app=ds-nginx"

# Chi tiết về ds
kubectl describe ds/dsapp

# Xóa DaemonSet
kubectl delete ds/dsapp

# xóa taint trên node master.xtl cho phép tạo Pod
kubectl taint node master.xtl node-role.kubernetes.io/master-

# thêm taint trên node master.xtl ngăn tạo Pod trên nó
kubectl taint nodes master.xtl node-role.kubernetes.io/master=false:NoSchedule
```
#Job trong Kubernetes
- job có chức năng tạo các POD đảm bảo nó chạy và kết thúc thành công trong 1 khoảng thời gian. Khi hoàn thành thì POD kết thúc và dừng.
- Khi xóa job thì các POD nó tạo ra cũng sẽ xóa theo. Một job có thể tạo ra các POD chạy tuần tự hoặc song song
- sử dụng job khi muốn thi hành một vài chức năng hoàn thành xong thì dừng lại (ví dụ: backup, update, kiểm tra,...)

- Khi job tạo POD, POD chưa hoàn thành nếu POD bị xóa hoặc lỗi node thì nó sẽ thực hiện tạo POD khác để thi hành tác vụ

## ví dụ:
```
apiVersion: batch/v1
kind: Job
metadata:
  name: myjob
spec:
  # Số lần chạy POD thành công
  completions: 10
  # Số lần tạo chạy lại POD bị lỗi, trước khi đánh dấu job thất bại
  backoffLimit: 3
  # Số POD chạy song song
  parallelism: 2
  # Số giây tối đa của JOB, quá thời hạn trên hệ thống ngắt JOB
  activeDeadlineSeconds: 120

  template:
    spec:
      containers:
      - name: busybox
        image: busybox
        command:
          - /bin/sh
          - -c
          - date; echo "Job executed"
      restartPolicy: Never
```
** Triển khai 1 job
```kubectl apply -f 2.job.yaml```

**Thông tin job có tên myjob
```kubectl describe job/myjob```

#CronJob trong kubernetes
- Cronjob: chạy các job theo lịch đặt sẵn. Việc khai báo giống như sử dụng Crontab trên Linux
- Hữu ích khi tạo các task định kỳ/lặp lại: backup, send email,...
- CronJobs cũng có thể lên lịch cho các task riêng lẻ trong một thời gian cụ thể, chẳng hạn như lên lịch cho một job khi cluster có khả năng không hoạt động.

## ví dụ:
```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: mycronjob
spec:
  # Một phút chạy một Job
  schedule: "* * * * *"
  # Số Job lưu lại, để xem lịch sử hoạt động
  successfulJobsHistoryLimit: 3
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: busybox
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo "Job in CronJob"
          restartPolicy: Never
```
** Danh sách các CronJob
```kubectl get cj -o wide```

** Danh sách các Job
```kubectl get jobs -o wide```