# Dashboard安装

```shell
# 安装dashboard，国内可以使用别的yaml源
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml

# 修改node为NodePort模式
kubectl patch svc -n kube-system kubernetes-dashboard -p '{"spec":{"type":"NodePort"}}'

# 查看服务(得知dashboard运行在443:32383/TCP端口)
kubectl get svc -n kube-system 
# --- 输出 ---
NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
kube-dns               ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP,9153/TCP   7h40m
kubernetes-dashboard   NodePort    10.111.77.210   <none>        443:32383/TCP            3h42m
# --- 输出 ---

# 查看dashboard运行在哪个node(得知dashboard运行在192.168.20.4)
kubectl get pods -A -o wide
# --- 输出 ---
NAMESPACE     NAME                                    READY   STATUS    RESTARTS   AGE     IP             NODE     NOMINATED NODE   READINESS GATES
kube-system   coredns-fb8b8dccf-rn8kd                 1/1     Running   0          7h43m   10.244.0.2     master   <none>           <none>
kube-system   coredns-fb8b8dccf-slwr4                 1/1     Running   0          7h43m   10.244.0.3     master   <none>           <none>
kube-system   etcd-master                             1/1     Running   0          7h42m   192.168.20.5   master   <none>           <none>
kube-system   kube-apiserver-master                   1/1     Running   0          7h42m   192.168.20.5   master   <none>           <none>
kube-system   kube-controller-manager-master          1/1     Running   0          7h42m   192.168.20.5   master   <none>           <none>
kube-system   kube-flannel-ds-amd64-l8c7c             1/1     Running   0          7h3m    192.168.20.5   master   <none>           <none>
kube-system   kube-flannel-ds-amd64-lcmxw             1/1     Running   1          6h50m   192.168.20.4   node1    <none>           <none>
kube-system   kube-flannel-ds-amd64-pqnln             1/1     Running   1          6h5m    192.168.20.3   node2    <none>           <none>
kube-system   kube-proxy-4kcqb                        1/1     Running   0          7h43m   192.168.20.5   master   <none>           <none>
kube-system   kube-proxy-jcqjd                        1/1     Running   0          6h5m    192.168.20.3   node2    <none>           <none>
kube-system   kube-proxy-vm9sj                        1/1     Running   0          6h50m   192.168.20.4   node1    <none>           <none>
kube-system   kube-scheduler-master                   1/1     Running   0          7h42m   192.168.20.5   master   <none>           <none>
kube-system   kubernetes-dashboard-5f7b999d65-2ltmv   1/1     Running   0          3h45m   10.244.1.2     node1    <none>           <none>
# --- 输出 ---
# 如果无法变成Running状态，可以使用以下命令排错
journalctl -f -u kubelet  # 只看当前的kubelet进程日志
### 提示拉取镜像失败，无法翻墙的可以使用以下方法预先拉取镜像
### 请在kubernetes-dashboard的节点上操作：
docker pull mirrorgooglecontainers/kubernetes-dashboard-amd64:v1.10.1
docker tag mirrorgooglecontainers/kubernetes-dashboard-amd64:v1.10.1 k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.1
docker rmi  mirrorgooglecontainers/kubernetes-dashboard-amd64:v1.10.1
```

根据上面的信息可以得知dashboard的ip和端口，使用**火狐浏览器**或最新版本**谷歌浏览器**访问https://192.168.200.25:32383（必须使用**https**）

**注意：如果是centos7，需要关闭iptables或者增加规则**
```
# 修改启动脚本
vim /lib/systemd/system/docker.service
# 在[Service]下添加规则
ExecStartPost=/sbin/iptables -I FORWARD -s 0.0.0.0/0 -j ACCEPT 
# 重启docker，即可访问到dashboard
systemctl restart docker
```

![登录dashboard](/home/freeze/Document/gitbook/static/1555422013719.png)

```sh
# 创建dashboard管理用户
kubectl create serviceaccount dashboard-admin -n kube-system

# 绑定用户为集群管理用户
kubectl create clusterrolebinding dashboard-cluster-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin

# 生成tocken
kubectl describe secret -n kube-system dashboard-admin-token
# --- 输出如下 ---
Name:         dashboard-admin-token-pb78x
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: dashboard-admin
              kubernetes.io/service-account.uid: 166aeb8d-604e-11e9-80d6-080027d8332b

Type:  kubernetes.io/service-account-token


Data(qxl:done)
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      
eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkYXNoYm9hcmQtYWRtaW4tdG9rZW4tbHBzc2oiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGFzaGJvYXJkLWFkbWluIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiOGUxNzM3YjUtNjE3OC0xMWU5LWJlMTktMDAwYzI5M2YxNDg2Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmRhc2hib2FyZC1hZG1pbiJ9.KHTf4_3DJu0liKeoOIoCssmIRXSHM_A4w9XVJKQ44jqEfPSbpwohqKnHxOspWAWsjwRrc3kSQyC9KEDCfTYl91ZY_PzUSqPG8XY58ab1p9q1xUxdDYu3qCyaSHWTQ2dATl1G5nNZQLfrarwWIPurm0BLBLsR1crIQj1P8VGafJJXz-TCQZgiw1OHqB8w89IBUhGrn8vuaIdspNLNZmrl-icjFS4eAevBREwlxqxX0-3-mzTFE8xqCHyfJ7pKpK-Jv1jSpuHjb0CfDPvNBuAGp5jQG44Ya6wq1BcqQO4RiQ07hjfIrnwmfWyZWmBn9YLvBVByupLv872kUUSSxjxxbg
# ------
```

使用生成的tocken就可以登录dashboard了。

![dashboard概况](/home/freeze/Document/gitbook/static/1555423252759.png)
