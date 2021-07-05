##Install k8s cluster with kubeadm (3 node, 1 master - 2 worker)

#cấu hình file hosts trên 3 node
cat  <<EOF >> /etc/hosts
192.168.88.130 k8s-master01
192.168.88.131 k8s-worker01
192.168.88.132 k8s-worker02
EOF

#install docker version 19.03.15 trên cả 3 node

echo   "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
	  
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
apt update
apt-get install docker-ce=5:19.03.15~3-0~ubuntu-bionic docker-ce-cli=5:19.03.15~3-0~ubuntu-bionic containerd.io
systemctl start docker

#cấu hình docker daemon
cat  <<EOF >> /etc/docker/daemon.json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "50m",
    "max-file": "7",
    "compress": "true"
  }
}
EOF

systemctl restart docker

# Install kubeadm, kubelet, kubectl (version =1.18.19-00 do version mới không hỗ trợ cri docker)
# Trên các 3 node
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"

# Trên Master
apt-get install -qy kubelet=1.18.19-00 kubectl=1.18.19-00 kubeadm=1.18.19-00

# Trên worker
apt-get install -qy kubelet=1.18.19-00 kubectl=1.18.19-00

# Disable swap, thực hiện trên cả 3 VM
swapoff -a

#Letting iptables see bridged traffic
## Trên 3 node
modprobe br_netfilter
 
## Setup required sysctl params, these persist across reboots.
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
 
## Apply sysctl params without reboot
sudo sysctl --system

# Init cluster bằng kubeadm, thực hiện trên Master
kubeadm init --pod-network-cidr=10.244.0.0/16 --cri-socket=/run/containerd/containerd.sock #sử dụng dải 10.244.0.0/16 cho pod nếu sử dụng flannel network plugin, nếu sử dụng calico thì dùng dải 192.168.0.0/16
 
# Sau khi init thành công, thực hiện join các Worker vào cluster, thực hiện các bước trên các Worker
kubeadm join [Master IP]:6443 --token <token> \
--discovery-token-ca-cert-hash sha256:<hash> --cri-socket=/run/containerd/containerd.sock # token và ca cert hash sẽ có thể lấy được ngay sau khi init thành công cluster