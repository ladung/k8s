Khi một ứng dụng quan trọng đối với sản phẩm của bạn và bạn muốn đảm bảo rằng các phiên bản mới của ứng dụng này luôn được scheduled (lên lịch triển khai) trước, ngay cả khi cụm Kubernetes của bạn đang chịu áp lực về tài nguyên. Một giải pháp thường được nghĩ đến đầu tiền để giải quyết vấn đề này là luôn cung cấp quá mức tài nguyên cho cụm của bạn để luôn có một số tài nguyên chờ sẵn cho các tình huống mở rộng quy mô. Giải pháp này thường hoạt động, nhưng chi phí cao hơn vì bạn sẽ phải trả tiền cho các tài nguyên mà phần lớn thời gian chúng đều nhàn rỗi. Tuy nhiên, còn có một giải pháp khác là sử dụng *"Pod Priority và Preemption"*.

Vậy Pod Priority và Preemption là gì và được sử dụng như thế nào, chúng ta sẽ cùng tìm hiểu trong bài viết này nhé.

Priority Pod và Preemption là gì ?
==================================

Pod priority and preemption hay Mức độ ưu tiên của pod và quyền ưu tiên pod là một tính năng scheduled (lập lịch) cho pod được cung cấp stable kể từ phiên bản Kubernetes 1.14 cho phép bạn đạt được mức độ ưu tiên lập lịch mà bạn mong muốn cho các pod chứa các ứng dụng quan trọng của mình mà không cần cung cấp quá nhiều tài nguyên chờ sẵn cho các cụm. Nó cũng là một cách để cải thiện việc sử dụng tài nguyên trong các cụm của bạn mà không làm mất đi sự ổn định của các khối lượng công việc thiết yếu được triển khai trong cụm.

-   Pod priority (mức độ ưu tiên của pod): Cho biết mức độ quan trọng của một pod so với các pod khác và xếp hàng các pod theo thứ tự dựa trên mức độ ưu tiên.
-   Pod preemption (quyền ưu tiên pod): Cho phép cụm Kubernetes của bạn loại bỏ (evict) hoặc cố gắng loại bỏ (preempt) pod có mức độ ưu tiên thấp hơn để các pod có mức độ ưu tiên cao hơn có thể được lập lịch nếu không còn chỗ trống trên một node thích hợp.

PriorityClass
=============

Để có thể sử dụng Pod priority và preemption thì trước hết, chúng ta cần nhắc đến khái niệm PriorityClass:

-   Một `PriorityClass` là một *non-namespaced object* và nó là một phần của bộ lập lịch được sử dụng để xác định ánh xạ từ tên của một priority class đến một giá trị priority pod. Các giá trị là là một giá trị số nguyên và nó càng lớn thì mức độ ưu tiên càng cao. Giá trị này được xác định trong trường một trường bắt buộc của PriorityClass là `value`.
-   `PriorityClass` có thể có bất kỳ giá trị số nguyên 32 bit nào nhỏ hơn hoặc bằng 1 tỷ. Các số lượng lớn hơn được dành riêng cho các pod hệ thống quan trọng mà thường không được loại bỏ hay chiếm trước.
-   Tên của một `PriorityClass` phải có giá trị là một [DNS subdomain](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#dns-subdomain-names) và không được bắt đầu bằng `system-`
-   Trường tùy chọn `globalDefault` để chỉ ra rằng giá trị của PriorityClass này sẽ được sử dụng cho pod không có trường `priorityClassName`. Chỉ một PriorityClass duy nhất `globalDefault` được đặt thành `true` có thể tồn tại trong hệ thống. Nếu không có PriorityClass nào có `globalDefault`, mức độ ưu tiên của pod không có trường `priorityClassName` là 0.
-   Trường tuỳ chọn `description` là một chuỗi tùy ý. Nó được dùng để thêm mô tả chi tiết về PriorityClass để cho người dùng của cụm biết khi nào họ nên sử dụng PriorityClass này.
-   Ngoài ra, PriorityClass có một trường được đặt tên `PreemptionPolicy` để xác định hành vi của nó tương ứng với quyền ưu tiên. Theo mặc định, các giá trị của nó là `PreemptionPolicy=PreemptLowerPriority`, sẽ cho phép các pod của PriorityClass đó ưu tiên các pod có mức độ ưu tiên thấp hơn. Nếu `PreemptionPolicy=Never` các pod trong PriorityClass đó sẽ không ưu tiên các pod khác.

Cách sử dụng pod priority và preemption
=======================================

### Tạo một hoặc nhiều `PriorityClass`

Ví dụ ta có thể tạo một số PriorityClass mẫu như sau:

```
---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority-preempting
value: 1000000
preemptionPolicy: PreemptLowerPriority
globalDefault: false
description: "This priority class will cause other lower priority pods to be preempted."
---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority-nonpreempting
value: 1000000
preemptionPolicy: Never
globalDefault: false
description: "This priority class will not cause other pods to be preempted."
---

```

### Tạo các pod priority

Sau khi đã tạo một hoặc nhiều PriorityClass, ta sẽ có thể tạo các pod được ưu tiên bằng cách thêm trường `priorityClassName` với giá trị là tên của một PriorityClass. Ví dụ:

```
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-preempting
  labels:
    env: test
spec:
  containers:
  - name: nginx-preempting
    image: nginx-preempting
    imagePullPolicy: IfNotPresent
  priorityClassName: high-priority-preempting
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-nonpreempting
  labels:
    env: test
spec:
  containers:
  - name: nginx-nonpreempting
    image: nginx-nonpreempting
    imagePullPolicy: IfNotPresent
  priorityClassName: high-priority-nonpreempting

```

Tạm kết
=======

Sử dụng Pod Priority và Preemption là một phương pháp khá hay bạn có thể sử dụng để cải thiện việc sử dụng tài nguyện cụm của mình, bạn cũng có thể giúp ngăn khối lượng công việc có mức độ ưu tiên thấp hơn ảnh hưởng đến khối lượng công việc quan trọng trong cụm của bạn, đặc biệt là trong trường hợp cụm bắt đầu đạt đến giới hạn khả năng tài nguyên của nó.
