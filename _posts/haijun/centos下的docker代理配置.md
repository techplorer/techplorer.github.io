这几天频繁的给同事分配虚拟机、装系统、安装软件，涉及docker的代理配置。因为我们的工作机所在的网络与互联网是隔开的，所以要想让后台的服务器联网，需要走代理。但每次配置时都得用一下搜索引擎，比较繁琐，这里就简单记录一下基本的命令，方便后续配置操作的查找。



### docker版本拉到最新

docker版本拉取到最新，不然容易踩坑，比如我今天用1.13.1的docker版本，当配置完代理后，使用docker去拉取镜像时就会报错：

![image.png](images/centos下的docker代理配置-1.png)



有意思的是当我使用wget去fetch网页数据时，是可以通的(因为在此之前已经设置过http代理了，所以wget是可以走代理服务器去联网的)

![image.png](images/centos下的docker代理配置-2.png)

从上面的wget与docker.io的交互中可以发现，中间有多次与代理服务的网络请求交互，所以我猜测之所以docker不能正常使用代理，可能是docker版本比较低有关系，可能有点小bug在里面。



这里也就不要折腾了，意义不是很大，直接使用如下命令把docker软件版本升到最新版本：

- 暴力一点，直接把docker相关的软件统统卸载干净

- 安装docker软件

- 重启docker相关的所有服务

  ```bash
  sudo yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-selinux  docker-engine-selinux docker-engine
  
  sudo yum install -y yum-utils device-mapper-persistent-data lvm2
  sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
  sudo yum install docker-ce
  
  sudo systemctl start docker
  sudo systemctl enable docker
  sudo systemctl status docker
  ```

### 配置docker代理

- 创建一个docker配置文件

- 把代理服务的配置按格式填充

- 最后重启一下docker服务

  ```bash
  mkdir -p /etc/systemd/system/docker.service.d
  vim  /etc/systemd/system/docker.service.d/http-proxy.conf
  
  [Service]
  Environment="HTTP_PROXY=http://proxy.example.com:80/"
  
  systemctl daemon-reload
  service docker restart
  ```



至此，我们就搞定了docker代理的问题了，测试一下：

![image.png](images/centos下的docker代理配置-3.png)

再拉取一下镜像：

### ![image.png](images/centos下的docker代理配置-4.png)

好了，说明docker通过代理上网已经成功了。



### 资料

- https://docs.docker.com/engine/install/centos/
- https://stackoverflow.com/questions/26472586/upgrade-docker-on-centos-7
- https://linoxide.com/upgrade-docker-fedora-centos/

- https://stackoverflow.com/questions/58841014/set-proxy-on-docker