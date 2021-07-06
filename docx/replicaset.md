# POD
- Trong kubernetes thì pod là đơn vị nhỏ nhất. Pod được cấu thành từ 1 container hoặc nhiều container, chúng chia sẻ địa chỉ IP giữa các container với nhau. Nói cách khác nếu chúng ta tạo một Pod chứa 2 container trở lên thì các container này sẽ đồng nhất với nhau địa chỉ IP. Vì vậy các container trong cac pod sẽ trao đổi với nhau như trong môi trường local. Ngoài ra các container trong cùng 1 pod cũng chia sẻ với nhau vùng chứa gọi là volume

