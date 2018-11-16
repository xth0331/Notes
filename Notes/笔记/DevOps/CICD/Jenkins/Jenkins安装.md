



#  Jenkins 安装

Jenkins通常使用内置的`java servlet`容器/应用服务器（`Jetty`）在其自己的进程中作为独立应用程序运行。

##  硬件要求

| 规模   | RAM   | Drive Space                                 |
| ------ | ----- | ------------------------------------------- |
| 最低   | 256MB | 1GB（如果作为Dcoker容器运行，建议最小10GB） |
| 小团队 | 1GB+  | 50GB+                                       |



> Docker大法好

## 安装平台

### Docker

#### Linux/Mac

首先安装Docker，这里不介绍安装。

```bash
docker run \
  -u root \
  --rm \
  -d \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins-data:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkinsci/blueocean
```

这里的`-v jenkins-data:/var/jenkins_home` 我解释一下，`-v`参数是将容器中的`/var/jenkins_home`映射到名字为`jenkins-data`的 *docker volume*，如果此docker volume不存在会自动创建，可以将`jenkins-data`替换为宿主机本地目录，如`$HOME/jenkins`。

#### Windows

同样，安装Docker，然后参数基本是一样的，docker的相关参数请参考docker文档，

```powershell
docker run ^
  -u root ^
  --rm ^
  -d ^
  -p 8080:8080 ^
  -p 50000:50000 ^
  -v jenkins-data:/var/jenkins_home ^
  -v /var/run/docker.sock:/var/run/docker.sock ^
  jenkinsci/blueocean
```

## 访问Jenkins

容器运行起来之后，就可以通过映射到宿主机的端口来访问了，如果你想进入jenkins容器内部，比如查看个初始密码，可以运行如下命令进入jenkins容器的shell：

```bash
docker ps # 找到jenkins容器CONTAINER ID,
docker exec -it JENKINS_CONTAINER_ID /bin/bash 
```

当然，也可以通过如下命令查看日志获得初始密码：

```bash
docker logs JENKINS_CONTAINER_ID
```

浏览器访问 宿主机IP+PORT（这里端口是8080）.



##　获取Administrator密码

- 进入容器，然后查看文件内容

```bash
docker exec -it JENKINS_CONTAINER_ID /bin/bash 
cat /var/jenkins_home/secrets/AdminPassword
```

- 通过`docker logs`查看

```bash
docker logs JENKINS_CONTAINER_ID
```

