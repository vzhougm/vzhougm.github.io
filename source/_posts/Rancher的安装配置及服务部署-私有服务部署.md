---
title: Rancher的安装配置及服务部署+私有服务部署
date: 2020-08-31 12:49:16
comments: true
categories:
 - 容器
tags: 
 - docker
 - rancher
---
# 1. 官网文档
---

https://rancher.com/docs/rancher/v2.x/en/quick-start-guide/deployment/quickstart-manual-setup/
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190628175056461.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70)
**Install Rancher**

```
sudo docker run -d --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher:stable 
```

**配置 https 安全证书**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190628190009509.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70)

    docker run -d --restart=unless-stopped \
     -p 80:80 -p 443:443 \
     -v /<CERT_DIRECTORY>/<FULL_CHAIN.pem>:/etc/rancher/ssl/cert.pem \
     -v /<CERT_DIRECTORY>/<PRIVATE_KEY.pem>:/etc/rancher/ssl/key.pem \
     <REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher:<RANCHER_VERSION_TAG> --no-cacerts

# 2. 准备
 
## 系统环境
---
操作系统： CentOS 7.6 64位
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190703144411409.png)
## Docker 安装
---
参考 [CentOS 7 下 yum 安装 Docker CE](https://blog.csdn.net/GMingZhou/article/details/94024453)

<br>

## vsftpd文件上传服务
----

**根据自己实际需要安装**，这是我之前制作docker镜像时用于传输一些配置文件用的，一起记录下来了而已。

    sudo docker run -d -p 20-21:20-21 -p 21100-21110:21100-21110 -e FTP_PASS=123 -v /home/anrj/upload:/home/vsftpd --name vsftpd fauria/vsftpd

## docker命令不需要敲sudo的方法
---

**创建一个docker组**

```
sudo groupadd docker
```
**添加当前用户到docker组**

```
sudo usermod -aG docker $USER
```
登出，重新登录shell

<br>

# 3. 安装 Rancher
---

    docker run -d --restart=unless-stopped \
      -p 8080:80 -p 4433:443 \
      -v /opt/rancher:/var/lib/rancher \
      -v /var/log/rancher/auditlog:/var/log/auditlog \
      -v /home/anrj/ssl/cert.pem:/etc/rancher/ssl/cert.pem \
      -v /home/anrj/ssl/key.pem:/etc/rancher/ssl/key.pem \
      -e AUDIT_LEVEL=3 \
      --name rancher_stable \
      rancher/rancher:stable --no-cacerts;

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190703110956200.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70)
# 4. 添加集群
---
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190703111050449.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190703111133416.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70)
## 配置主机节点
---
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190703145436149.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70)
## 复制docker 运行命令，点击完成。在主机上执行，然后等待注册
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190703111653233.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190703111740702.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70)
## 主机注册成功
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190703112014679.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70)

# 5. 放几个自己常用的配置
**无实际作用**，仅个人使用，记录而已！

---

## docker-registry
registry:latest
5000
/home/anrj/docker/data/registry
/var/lib/registry

## mongo
27017
MONGO_INITDB_ROOT_USERNAME=root
MONGO_INITDB_ROOT_PASSWORD=123456

/home/anrj/docker/data/mongo/data
/data/db

## mysql
mysql:latest
3306
MYSQL_ROOT_PASSWORD = 123456

/home/anrj/docker/data/mysql
/var/lib/mysql

## redis
redis
6379
/home/anrj/docker/data/redis/
/data

## jrebel-reverse-proxy
127.0.0.1:50/jrebel_reverse_proxy
8888

## vz-earlyup
127.0.0.1:50/vz-earlyup:latest
9099

<br>

# 6. 添加镜像库凭证
---
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190703125837918.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70)这样就可以部署`自己的`项目了~

<br>

# 7. 部署服务
---
## 通用服务
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190703130742198.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70)
## 自己的服务
<br>

### 制作docker镜像
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019070313211656.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70)
### 上传至仓库

**IDEA操作**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190703132229609.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70)
#### 上传之前清理一下之前上传的镜像
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190703132840529.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70)
**由于仓库的端口号变了，之前的镜像没用了，所以先清理一下**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190703133001944.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70)
![](https://img-blog.csdnimg.cn/20190703132350339.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190703132443482.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70)
#### 踩坑1
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190703133939710.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70)
#### 踩坑2
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190703134238779.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70)
本地镜像仓库端口变了，但设置里没改过来！

    sudo vim /etc/docker/daemon.json
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190703134646264.png)

    sudo systemctl daemon-reload
    sudo service docker restart

**but IDEA里操作还是推送失败！** ?‍♂️

**还是直接命令操作吧**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190703135747334.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70)
**ok push 成功！** [IP]:[Port]/v2/_catalog
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190703135938443.png)

### 部署自己的服务
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190703131240813.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70)
**启动正常，访问正常~**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190703140418351.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70)
# 8. OK，Rancher 一套流程完毕~
---
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190703141637814.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70)

# 9. 最后再给大家两个实用脚本
---

## 重装
**rancher_reload.sh**
<font color="red">注意：此脚本会把所有的docker镜像给删除！</font>

    #!/bin/sh
    echo "重装Rancher"
    
    sudo docker rm -f $(sudo docker ps -qa)
    sudo docker rmi -f $(sudo docker images -q)
    sudo docker volume rm $(sudo docker volume ls -q)
    
    for mount in $(mount | grep tmpfs | grep '/var/lib/kubelet' | awk '{ print $3 }') /var/lib/kubelet /var/lib/rancher; do umount $mount; done
    
    cleanupdirs="/var/lib/etcd /etc/kubernetes /etc/cni /opt/cni /var/lib/cni /var/run/calico /opt/rke"
    for dir in $cleanupdirs; do
      echo "Removing $dir"
      rm -rf $dir
    done
    
    sudo rm -rf /opt/rancher /var/log/rancher/auditlog /etc/ceph /etc/cni /etc/kubernetes /opt/cni /opt/rke /run/secrets/kubernetes.io /run/calico /run/flannel /var/lib/calico /var/lib/etcd /var/lib/cni /var/lib/kubelet /var/lib/rancher/rke/log /var/log/containers /var/log/pods /var/run/calico
    
    iptables -F && iptables -t nat -F
    ip link del flannel.1
    ip link del tunl0
    
    sudo systemctl daemon-reload
    sudo systemctl restart docker
    
    echo "清理完毕~"
    echo ""
    echo "开始安装..."
    
    docker run -d --restart=unless-stopped \
      -p 8080:80 -p 4433:443 \
      -v /opt/rancher:/var/lib/rancher \
      -v /var/log/rancher/auditlog:/var/log/auditlog \
      -v /home/anrj/ssl/cert.pem:/etc/rancher/ssl/cert.pem \
      -v /home/anrj/ssl/key.pem:/etc/rancher/ssl/key.pem \
      -e AUDIT_LEVEL=3 \
      --name rancher_stable \
      rancher/rancher:stable --no-cacerts;
    
    echo "重装完毕~"

## 私有仓库镜像删除

    curl https://raw.githubusercontent.com/burnettk/delete-docker-registry-image/master/delete_docker_registry_image.py | sudo tee /usr/local/bin/delete_docker_registry_image >/dev/null
    sudo chmod a+x /usr/local/bin/delete_docker_registry_image

参考 https://segmentfault.com/a/1190000011153919

# 10. 结语
折腾吧！不行就推到重来，毕竟上面两个脚本就是为折腾而生的~ ?