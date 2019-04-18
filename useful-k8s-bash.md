```bash
journalctl -xeu kubelet  

kubeadm reset

kubeadm init  --apiserver-advertise-address=192.168.200.24  --kubernetes-version v1.13.5  --service-cidr=10.1.0.0/16  --pod-network-cidr=10.244.0.0/16

systemctl status kube-apiserver.service

netstat -lnp|grep 10250

ps -ef|grep kube  
```


