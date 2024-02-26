# Centos环境下安装docker
## 移除以前docker相关包
```
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```
## 配置yum源
```
sudo yum install -y yum-utils
sudo yum-config-manager \
--add-repo \
http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

## 安装docker
```
yum install -y docker-ce-20.10.7 docker-ce-cli-20.10.7  containerd.io-1.4.6
```
## 启动docker
```
systemctl enable docker --now
```
## 配置镜像加速
可以自己去阿里云查询自己的镜像服务地址  

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://82m9ar63.mirror.aliyuncs.com"],
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```
