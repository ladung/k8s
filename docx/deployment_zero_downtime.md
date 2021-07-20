
# Zero downtime với k8s deployment

* Ví dụ *
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name:  ${CI_PROJECT_NAME}
  namespace: $NAMESPACE
  labels:
    name:  $CI_PROJECT_NAME
    app: ${CI_PROJECT_NAME}
spec:
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  selector:
    matchLabels:
      app: ${CI_PROJECT_NAME}
  template:
    metadata:
      labels:
        name:  $CI_PROJECT_NAME
        app: ${CI_PROJECT_NAME}
    spec:
      containers:
      - image:  asia.gcr.io/ctel-rnd/$CI_PROJECT_NAME:$IMAGE_TAG
        name:  $CI_PROJECT_NAME
        livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 3
            failureThreshold: 3
        readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 3
            failureThreshold: 3
        # envFrom:
        # # - configMapRef:
        # #     name: $CI_PROJECT_NAME
        # - secretRef:
        #     name: $CI_PROJECT_NAME
        # - secretRef:
        #     name: backend-common       
        ports:
        - containerPort:  80
        imagePullPolicy: Always
      nodeSelector:
        environment: ${APP_ENV}
        role: apps
      imagePullSecrets:
        - name: ctel-rnd-gcr-secret
      restartPolicy: Always
---
kind: Service
apiVersion: v1
metadata:
  name:  ${CI_PROJECT_NAME}
  namespace: $NAMESPACE
spec:
  selector:
    app:  ${CI_PROJECT_NAME}
  type:  ClusterIP
  ports:
  - protocol: TCP
    port:  80
    targetPort:  80
---
kind: Ingress
apiVersion: networking.k8s.io/v1beta1
metadata:
  name: ${CI_PROJECT_NAME}
  namespace: $NAMESPACE
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
  - host: $HOST
    http:
      paths:
      - path: ${SERVICE_NAME}(/|$)(.*)
        backend:
          serviceName: ${CI_PROJECT_NAME}
          servicePort: 80
```

Trong đó:
- Deployment đang đảm bảo thay thế các Pods cũ bằng các Pod mới dần dần trong khi vẫn giữ một phần trong số các Pod chạy. Phần này có thể được kiểm soát bằng cách thay đổi các tham số maxSurge và maxUnavailable.

  - maxSurge: số lượng Pod có thể được triển khai tạm thời bên cạnh các bản sao mới. Setting thông số này có nghĩa là chúng ta có thể có tổng cộng tối đa năm Pod đang chạy trong quá trình cập nhật (4 bản sao + 1).
  - maxUnavailable: số lượng Pod có thể bị kill đồng thời trong quá trình update. Trong ví dụ của chúng tôi, chúng tôi có thể có ít nhất ba Pod running trong khi quá trình update đang diễn ra (4 bản sao - 1).
## Tham khảo
- https://kipalog.com/posts/Zero-downtime-voi-Kubernetes-P1--Truly-stateless-application
- https://blog.vietnamlab.vn/kubernetes-best-pratice-zero-downtime-rolling-update/
- https://cuongquach.com/co-che-healthcheck-readiness-liveness-kubernetes.html
- https://blog.vietnamlab.vn/kubernetes-best-practice-p2-application-process-management-with-poststart-and-prestop-hook/
