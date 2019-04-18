# centos7搭建Harbor企业级docker仓库

### 安装docker

```shell
curl -fsSL "https://get.docker.com/" | sh
systemctl enable --now docker
```

### 安装docker-compose

```shell
yum update -y
yum install python-pip
pip install --upgrade setuptools  # 可能由于setuptools版本过低报错
pip install docker-compose  # 如果报错可以试试 --ignore-installed
```

### 安装Harbor

```shell
wget -P /usr/local/src/     https://storage.googleapis.com/harbor-releases/release-1.7.0/harbor-online-installer-v1.7.5.tgz  # 在线安装
# 最新版本请查看https://github.com/goharbor/harbor/releases/

cd /usr/local/src/
tar zxf harbor-online-installer-v1.7.5.tgz  -C /usr/local/
cd /usr/local/harbor/
bash install.sh # 使用--with-clair添加镜像漏洞扫描功能
```

### 配置文件

```shell
vim /usr/local/harbor/harbor.cfg  # harbor配置文件

# 找到以下项目并且修改
hostname = test.com  # 修改访问域名，如果使用其它端口，请在后面添加端口号，如test.com:8080
#邮箱配置（根据实际账号配置）
email_server = smtp.qq.com
email_server_port = 465
email_username = test@qq.com
email_password = 123456
email_from = test@qq.com  # 经测试发现必须要和email_username相同才可以发邮件
email_ssl = true  # 开启ssl保护，使用端口465，关闭使用端口25
#禁止用户注册
self_registration = off
#设置只有管理员可以创建项目
project_creation_restriction = adminonly
#设置管理员密码
harbor_admin_password = 123456
```

### 容器集群管理

```shell
cd /usr/local/harbor/
docker-compose ps  # 查看harbor集群容器，安装后已经启动

# ---------- 控制 ----------
# 必须要在/usr/local/harbor/目录下，或者-f指定docker-compose.yml
# 启动Harbor
docker-compose start
# 停止Harbor
docker-comose stop
# 重启Harbor
docker-compose restart
# 移除Harbor
docker-compose down -v  # -v 参数移除vloume
# 重新创建并启动
docker-compose up -d
# ---------- 控制 ----------
```

#### 修改nginx端口（如有需要）

```shell
vim /usr/local/harbor/docker-compose.yml
# 把proxy下的80:80改为8080:80则为使用8080访问harbor
docker-compose stop proxy  # proxy其实就是nginx
docker-compose up -d proxy  # 重新开启nginx
netstat -lntp # 查看本地打开端口，如果有docker-proxy为8080则修改成功
# 如果有安全组防火墙，记得先放行对应端口
```

### 访问网页

使用账号admin，默认密码Harbor12345，如果修改了配置文件的密码，则使用上面修改的密码。

![登录界面](/home/freeze/Document/gitbook/static/1555394389198.png)

+ 默认是所有人可以创建用户登录的，只是上面安装配置中禁止了用户注册。

![功能面板](/home/freeze/Document/gitbook/static/1555394437899.png)

![系统配置](/home/freeze/Document/gitbook/static/1555394503239.png)

+ 系统配置中可以设置邮箱配置，认证配置、垃圾清理等，但是不可以设置web打开的端口。

![漏洞扫描功能](/home/freeze/Document/gitbook/static/1555394579735.png)

+ 通过漏洞扫描，可以分析出镜像存在的一些漏洞缺陷编码，并且提供修复建议。

### 上传、下载镜像

```shell
# 由于使用80端口需要备案，harbor页面已经修改为8080端口（注意修改harbor.cfg的hostname后需要重新执行install.sh）
vim /etc/docker/daemon.json
# 添加 "insecure-registries":["test.com:8080"] }
docker login test.com:8080  # 尝试登录
# 编写dockerfile
mkdir ~/test_harbor && cd ~/test_harbor
cat << EOF > Dockerfile
FROM nginx:latest
MAINTAINER test "test@qq.com"
# 配置环境变量
ENV LANG=C.UTF-8 TZ=Asia/Shanghai
EOF
# build镜像
docker build -t test.com:8080/library/nginx:latest .
# push镜像到远程仓库
docker push test.com:8080/library/nginx:latest
# 从远程仓库拉取镜像
docker pull test.com:8080/library/nginx:latest
```

[参考链接](https://www.cnblogs.com/pangguoping/p/7650014.html)

[为什么有了Docker registry还需要Harbor？](https://blog.csdn.net/jessise_zhan/article/details/80130104)









