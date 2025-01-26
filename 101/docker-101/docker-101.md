[toc]

# docker 的作用

- 将<u>*运行环境、软件、目录*</u>等打包，快速在其他机器上搭建起来



# 镜像

## 查看镜像

```cmd
docker images
```

- `REPOSITORY`：镜像名
- `TAG`：标签 (也可以看做版本号)



## 导入镜像

### 从远程仓库导入

```cmd
docker pull REPOSITORY:TAG
e.g. docker pull centos:latest
```

### 从本地导入

```cmd
docker load -i [path/to/image.tar]
e.g. docker load -i /home/docker-images/gitlab-ce-17.0.1.tar
```



## 导出镜像

### 从已有容器中导出镜像

```cmd
docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]
docker commit -m "Added a new file" -a "Author Name" abcdef123456 myimage:latest
```

- abcdef123456 代表**容器的 id**，将从这个容器中导出镜像
- myimage:latest 代表**容器的名字**和**标签**



### 使用 DockerFile 导出镜像

1. 创建 **DockerFile**

   ```dockerfile
   FROM node:8.4
   COPY . /app
   WORKDIR /app
   RUN npm install --registry=https://registry.npm.taobao.org
   EXPOSE 3000
   ```

   - `FROM`：**继承**其它的镜像

   - `COPY`：默认<u>*从宿主机*</u>将指定目录或文件**复制**<u>*到容器内*</u>的指定路径下

   - `WORKDIR`：设置<u>*容器内的*</u>**工作路径**；默认 `WORKDIR` 设置为 / (i.e. 根目录，这个目录下有 root、etc 等子目录)

   - `RUN`：执行指定的指令

     > 因为设置了 `WORKDIR /app`，所以会在 /app 目录下执行指令

   - `EXPOSE`：将指定的端口暴露 (可以通过这个<u>*端口从外部访问容器*</u>)

   > docker 容器并不会自动导入 <u>*pip 或 conda 环境*</u>，是<u>*需要显式写在 DockerFile 中*</u>的

2. <u>*使用 DockerFile 构建镜像*</u>

   ```cmd
   docker image build -t REPOSITORY:TAG PATH
   e.g. docker image build -t koa-demo:0.0.1 .
   ```

   - ` -t`：指定镜像的名字

   - `PATH`：（1）会在 `PATH` 路径下查找 DockerFile（2）设置 DockerFile 中<u>*宿主机的工作路径*</u> (e.g. 上方指令设置的 `PATH` 为 `.`，i.e. 当前路径)

     > e.g. 上方 DockerFile 的 `COPY . /app` 中的 `.` (i.e. 宿主机的当前目录)，实际是 `PATH` 所指的路径





# 容器管理

## 查看容器相关信息

```cmd
docker ps -a
```

- `-a`：查看所有容器，包括<u>*已停止的*</u>容器
- 使用这个指令可以查看**容器的 ID，容器的名字，容器使用的镜像，容器的端口映射**



## 启动容器

```cmd
docker run -itd -p <宿主机端口>:<容器端口> --name DOCKER_NAME REPOSITORY:TAG
e.g. docker run -itd -p 25001:3000 --name my-nginx2 nginx
```

- `-i`，`-t`，`-d`：让<u>*容器保持运行状态*</u>

  > 如果镜像中没有长时间运行的指令，在<u>*执行完成后容器会自动退出*</u>；而这三个参数可以让没有类似指令的容器保持运行状态

- `--name`：设置容器的名字

- `-p`：设置端口映射

  - 如果为 22 端口设置了端口映射 (i.e. ssh 默认使用的端口)，且在<u>*容器内已经启动了 ssh 服务*</u>，那么就可以<u>*通过 ssh 从容器外访问容器*</u>

  > 假设设置了端口映射 `-p 25001:22`，在宿主机上就可以通过 `ssh -p 25001 127.0.0.1` 进入容器 (如果希望从远程访问，可以将 127.0.0.1 替换为<u>*宿主机的 ip*</u>)



## 进入容器

```cmd
docker exec -it <CONTAINER NAMES> /bin/bash 
e.g. docker exec -it my_container /bin/bash
```

- 表示<u>*以命令行交互的方式进入容器*</u>



## 在容器和宿主机/远程/容器间传递文件

- 可以<u>*使用 docker 指令*</u>传递

  > 即使没有暴露端口一样可以使用
  >
  > 可以在容器和容器之间直接传递数据

  ```cmd
  docker cp <HOST FILE PATH> <CONTAINER NAMES>:<CONTAINER FILE PATH>
  ```





# 其它

## ssh 服务

- 假设服务器 A 向服务器 B 发起 ssh 请求，服务器 A 会<u>*随机选取一个端口*</u>与服务器 B 的 22 端口进行通信 (<u>*22 为 ssh 服务的默认端口*</u>)

### 启动 ssh 服务

1. 使用 `ssh --version` <u>*查看是否安装了 ssh*</u>
2. 如果没有安装，使用 `apt install -y openssh-server` <u>*安装 ssh*</u>

3. 修改 `/etc/ssh/sshd_config`；注意，==不是== `/etc/ssh/ssh_config`

   设置以下项，

   ```cmd
   PermitRootLogin yes
   PasswordAuthentication yes
   ```

4. <u>*启动 ssh 服务器*</u>，`service ssh start` 或 `service ssh restart`

5. <u>*查看 ssh 服务是否顺利启动*</u>，可以使用 `netstat -tuln` 查看 22 端口是否处于监听或 `service ssh status`



### 设置免密登录

1. 在<u>*发起 ssh 请求的服务器上*</u>，执行 `ssh-keygen -t rsa` 指令

2. 默认在 `C:\Users\{user}\.ssh` (Windows 系统) 或 `~/.ssh` 下生成两个文件，

   私钥：`id_rsa`；公钥：`id_rsa.pub`

3. <u>*从发起请求的服务器上*</u>，将 `id_rsa.pub` <u>*移动到被请求服务器*</u>的名为 `~/.ssh/authorized_keys` 的文件中 (如文件或目录不存在，需要自行创建，并<u>*使用 `chmod` 指令增加必要的权限*</u>)

   > 一般地，`.ssh` 目录权限为 700，`authorized_keys` 文件权限为 600

4. 不需要重启 ssh 服务，若操作正确，已经可以实现**免密登录**



### `scp` 指令

- 利用 `ssh` 服务进行文件传输

  ```cmd
  scp [选项] [源文件或目录] [用户名@]主机名:目标路径
  e.g. scp -r -P 2222 /path/to/local username@192.168.1.100:/path/to/remote/directory/
  ```

  - `-P`：指定接收请求服务器的 ssh 服务使用的端口号 (e.g. 某个 docker 容器设置了 `-p 2222:22` 的端口映射，那么需要指定 `-P 2222`)；注意，==是大写 P==





# 参考资料

- [docker 入门中文教程](https://wangchujiang.com/docker-tutorial/index.html)

