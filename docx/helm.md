# Helm
Deploy một ứng dụng lên Kubernetes cluster - đặc biệt là các ứng dụng phức tạp - đòi hỏi việc tạo một loạt các resource của ứng dụng đó, ví dụ như Pod, Service, Deployment, ReplicaSet ... . Mỗi resource lại yêu cầu phải viết một file YAML chi tiết cho nó để deploy. Điều đó dẫn đến các thao tác CRUD(Create, Read, Update, Delete) trên một loạt các resource này trở nên phức tạp. Để giải quyết vấn đề này, Helm được ra đời.

Như Ubuntu có apt, Centos có yum, tương tự Helm đóng vai trò là một Package Manager cho Kubernetes. Việc cài đặt các resource Kubernetes sẽ được thông qua và quản lý trên Helm.



Hiện tại, Helm là project chính thức của hệ sinh thái Kubernetes và được quản lý bởi Cloud Native Computing Foundation.

Helm cung cấp một số tính năng cơ bản mà một package manager cần phải có như sau:

Cài đặt các resource và tự động cài đặt các resource phụ thuộc.

Update và rollback các resource đã release.

Cấu hình.

Pull các package từ các repository