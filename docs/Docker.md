

# Docker

> **由于每台服务器的运行环境不同，你写好的安装流程、部署脚本并不一定在每个服务器都能正常运行**，经常会出错。这就给系统的部署运维带来了很多困难
>
> `Docker`技术能够避免部署对服务器环境的依赖，减少复杂的部署流程

> `CentOS`的虚拟机中安装`Docker`，以下都是在虚拟机当中操作

## Linux环境搭建

### 安装`VMware`

* 这里就不展开了，百度跟着教程安装就行~

### 创建虚拟机

* 这里就不展开了，百度跟着教程安装就行~
* 注意：`Centos7`是比较常用的一个`Linux`发行版本，在国内的使用比例还是比较高的。所以我们采用`Centos7`，首先要下载一个`Centos7`的`iso`文件（这个也百度找一个下载），创建虚拟机过程中安装程序光盘映像文件会用到的~

<img src="/images/docker/image-20240626114101618.png" alt="image-20240626114101618" style="zoom:50%;" />

### 安装`Centos7`

* 选中`Install Centos 7` 后按下回车

<img src="/images/docker/image-20240626114531790.png" alt="image-20240626114531790" style="zoom:50%;" />

* 会提示我们按下`enter`键继续

<img src="/images/docker/image-20240626114626002.png" alt="image-20240626114626002" style="zoom:50%;" />

* 过一会儿后，会进入语言选择菜单，选择中文-简体中文，然后继续

<img src="/images/docker/image-20240626114654682.png" alt="image-20240626114654682" style="zoom: 50%;" />

* 接下来，会进入安装配置页面

  <img src="/images/docker/image-20240626114715958.png" alt="image-20240626114715958" style="zoom:50%;" />

* 鼠标向下滚动后，找到系统-安装位置配置

  <img src="/images/docker/image-20240626114742074.png" alt="image-20240626114742074" style="zoom:50%;" />

* 选择刚刚添加的磁盘，并点击完成

  <img src="/images/docker/image-20240626114802190.png" alt="image-20240626114802190" style="zoom:50%;" />

* 然后回到配置页面，这次点击《网络和主机名》

  <img src="/images/docker/image-20240626114823325.png" alt="image-20240626114823325" style="zoom:50%;" />

  1. 修改主机名为自己喜欢的主机名，不要出现中文和特殊字符，建议用`localhost`
  2. 点击应用
  3. 将网络连接打开
  4. 点击配置，设置详细网络信息

  <img src="/images/docker/image-20240626114854977.png" alt="image-20240626114854977" style="zoom:50%;" />

* 最好用一个截图软件，记住上图中的网络详细信息，接下来的配置要参考

  ![image-20240626114936193](/images/docker/image-20240626114936193.png)

* 点击配置按钮后，我们需要把**网卡**地址改为`静态IP`，这样可以避免每次启动`虚拟机IP`都变化。所有配置照你自己截图的网络信息填写

  <img src="/images/docker/image-20240626115041178.png" alt="image-20240626115041178" style="zoom:50%;" />

  > 上图中的四个信息参考你自己本地**以太网(ens33)网卡**的截图

* 最后，点击完成按钮

  <img src="/images/docker/image-20240626115126181.png" alt="image-20240626115126181" style="zoom:50%;" />

* 回到配置界面后，点击开始安装

  <img src="/images/docker/image-20240626115158376.png" alt="image-20240626115158376" style="zoom:50%;" />

* 接下来需要设置`root`密码

  <img src="/images/docker/image-20240626115219068.png" alt="image-20240626115219068" style="zoom:50%;" />

* 填写你要使用的`root`密码，然后点击完成

  <img src="/images/docker/image-20240626115240014.png" alt="image-20240626115240014" style="zoom:50%;" />

* 接下来，耐心等待安装即可

  <img src="/images/docker/image-20240626115300333.png" alt="image-20240626115300333" style="zoom:50%;" />

* 等待安装完成后，点击**重启**

  <img src="/images/docker/image-20240626115319103.png" alt="image-20240626115319103" style="zoom:50%;" />

* 耐心等待一段时间，不要做任何操作，虚拟机即可启动完毕

  <img src="/images/docker/image-20240626115413292.png" alt="image-20240626115413292" style="zoom:50%;" />

* 输入用户名`root`，然后点击回车，会要求你输入密码

  <img src="/images/docker/image-20240626115437288.png" alt="image-20240626115437288" style="zoom:50%;" />

* 此时你要输入密码，不过需要注意的是密码是**隐藏**的，输入了也看不见。所以放心输入，完成后回车即可

  <img src="/images/docker/image-20240626115503017.png" alt="image-20240626115503017" style="zoom:50%;" />

* 只要密码输入正确，就可以正常登录。此时可以用命令测试虚拟机网络是否畅通

  ```sh
  ping baidu.com
  ```

  * 如果看到这样的结果代表网络畅通：

  ![image-20240626115539493](/images/docker/image-20240626115539493.png)

  * 默认`ping`命令会持续执行，按下`CTRL `+ `C`后命令即可停止，此时环境就算是全部安装完成了

### 安装一个SSH客户端

* 在`VMware`界面中操作虚拟机非常不友好，所以一般推荐使用专门的`SSH`客户端，市面上常见的有：
  - `Xshell`：个人免费，商业收费。推荐使用
  - `Finshell`：基础功能免费，高级功能收费，基于Java，内存占用较高（在1个G左右）。不推荐
  - `MobarXterm`：基础功能免费、高级功能收费。开源、功能强大、内存占用低（只有`10m`左右），但是界面不太漂亮。推荐使用
* 我这里安装的是`Xshell7`

<img src="/images/docker/image-20240626113805295.png" alt="image-20240626113805295" style="zoom:50%;" />

* 如下就是连接上了，在客户端里面操作非常方便的~

  <img src="/images/docker/image-20240626115845063.png" alt="image-20240626115845063" style="zoom: 67%;" />

## Docker安装

### 卸载旧版

* 如果系统中已经存在旧的Docker，则先卸载

  ```shell
  yum remove docker \
      docker-client \
      docker-client-latest \
      docker-common \
      docker-latest \
      docker-latest-logrotate \
      docker-logrotate \
      docker-engine
  ```

### 配置Docker的yum库

* 首先要安装一个yum工具

  ```shell
  yum install -y yum-utils
  ```

* 安装成功后，执行命令，配置Docker的yum源 - 这里我们使用阿里云的

  ```shell
  yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
  ```

### 安装Docker

* 执行命令，安装Docke

  ```bash
  yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
  ```

### 启动和校验

```Bash
# 启动Docker
systemctl start docker

# 停止Docker
systemctl stop docker

# 重启
systemctl restart docker

# 设置开机自启
systemctl enable docker

# 执行docker ps命令，如果不报错，说明安装启动成功
docker ps
```

### 配置镜像加速

> 这里以阿里云镜像加速为例

#### 注册阿里云账号

> 首先访问阿里云网站：https://www.aliyun.com/，注册一个账号。

#### 开通镜像服务

* 在首页的产品中，找到阿里云的**容器镜像服务**

<img src="/images/docker/image-20240626131403388.png" alt="image-20240626131403388" style="zoom:50%;" />

* 点击后进入控制台

<img src="/images/docker/image-20240626131424638.png" alt="image-20240626131424638" style="zoom:50%;" />

* 首次可能需要选择立刻开通，然后进入控制台

#### 配置镜像加速

* 找到**镜像工具**下的**镜像加速器**

<img src="/images/docker/image-20240626131523075.png" alt="image-20240626131523075" style="zoom:50%;" />

* 页面向下滚动，即可找到配置的文档说明

  <img src="/images/docker/image-20240626131609913.png" alt="image-20240626131609913" style="zoom:50%;" />

## 快速入门

> 要想让`Docker`帮我们安装和部署软件，肯定要保证你的机器上有`Docker`，请查看 `Docker安装`

### 部署`MySQL`

> 传统方式部署`MySQL`，大概的步骤有：
>
> - 搜索并下载`MySQL`安装包
> - 上传至`Linux`环境
> - 编译和配置环境
> - 安装

* `Docker`安装，仅仅需要一步即可，在命令行输入下面的命令

  ```shell
  docker run -d \
    --name mysql \
    -p 3306:3306 \
    -e TZ=Asia/Shanghai \
    -e MYSQL_ROOT_PASSWORD=123 \
    mysql
  ```

* 运行效果如图：

![image-20240626132712616](/images/docker/image-20240626132712616.png)

> `MySQL`安装完毕！
>
> * 执行命令后，`Docker`会自动搜索并下载`MySQL`（这里下载的不是安装包，而是**镜像**），接着会自动运行`MySQL`
> * 完全不用考虑运行的操作系统环境，它不仅仅在`CentOS`系统是这样，在`Ubuntu`系统、`macOS`系统、甚至是装了`WSL`的`Windows`下，都可以使用这条命令来安装`MySQL`（如果是**手动安装，必须手动解决安装包不同、环境不同的、配置不同的问题**！）
>
> **镜像**
>
> * 镜像中不仅包含了`MySQL`本身，还包含了其运行所需要的**环境**、**配置**、**系统级函数库**，因此它在运行时就有自己**独立的环境**，就可以**跨系统运行**，也不需要手动再次配置环境了
>
> **容器**
>
> * 镜像中不仅包含了`MySQL`本身，还包含了其运行所需要的**环境**、**配置**、**系统级函数库**，因此它在运行时就有自己**独立的环境**，就可以**跨系统运行**，也不需要手动再次配置环境了。这套独立运行的隔离环境我们称为**容器**
>
> **总结**
>
> * 因此，`Docker`安装软件的过程，就是自动搜索下载镜像，然后创建并运行容器的过程



### 执行原理

* `Docker`是去哪里搜索和下载镜像的呢？这些镜像又是谁制作的呢？

  <img src="/images/docker/image-20240626133640313.png" alt="image-20240626133640313" style="zoom: 67%;" />

  > `Docker`本身包含一个后台服务，我们可以利用`Docker`命令告诉`Docker`服务，帮助我们快速部署指定的应用。
  >
  > `Docker`服务部署应用时，首先要去搜索并下载应用对应的镜像，然后根据镜像创建并允许容器，应用就部署完成了。

<img src="/images/docker/image-20240626134000758.png" alt="image-20240626134000758" style="zoom: 67%;" />





### 命令解读

* 我们来看一下安装`MySQL`的命令到底是啥意思

```shell
docker run -d \
  --name mysql \
  -p 3306:3306 \
  -e TZ=Asia/Shanghai \
  -e MYSQL_ROOT_PASSWORD=123 \
  mysql
```

> **解读**
>
> - `docker run -d` ：创建并运行一个容器，`-d`则是让容器以后台进程运行
> - `--name mysql ` :  给容器起个名字叫`mysql`，也可以叫别的（**需要唯一**！用于区分不同容器）
> - `-p 3306:3306` :  设置端口映射。（右边的`3306`是`MySQL`，左边是宿主机映射的，可以变）
>   - **容器是隔离环境**，外界不可访问。但是可以**将宿主机端口映射到容器内的端口**，当访问宿主机指定端口时，就是在访问容器内的端口了。
>   - 容器内端口往往是由容器内的进程决定，例如`MySQL`进程默认端口是`3306`，因此容器内端口一定是`3306`；而**宿主机端口则可以任意指定**，一般与容器内保持一致。
>   - 格式： `-p 宿主机端口:容器内端口`，示例中就是将宿主机的`3306`映射到容器内的`3306`端口
> - `-e TZ=Asia/Shanghai` : 配置容器内进程运行时的一些参数
>   - 格式：`-e KEY=VALUE`，`KEY`和`VALUE`都由容器内进程决定
>   - 案例中，`TZ=Asia/Shanghai`是设置时区；`MYSQL_ROOT_PASSWORD=123`是设置`MySQL`默认密码
> - `mysql` : 设置**镜像**名称，Docker会根据这个名字搜索并下载镜像
>   - 格式：`REPOSITORY:TAG`，例如`mysql:8.0`，其中`REPOSITORY`可以理解为镜像名，`TAG`是版本号
>   - 在未指定`TAG`的情况下，默认是最新版本，也就是`mysql:latest`
>
> **注意**
>
> * 镜像的名称不是随意的，而是要到`DockerRegistry`中寻找
> * 镜像运行时的配置也不是随意的，要参考镜像的帮助文档
> * 以上在`DockerHub`网站或者软件的官方网站中都能找到

## Docker基础

### 常见命令

| **命令**       | **说明**                       | **文档地址**                                                 |
| :------------- | :----------------------------- | :----------------------------------------------------------- |
| docker pull    | 拉取镜像                       | [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) |
| docker push    | 推送镜像到DockerRegistry       | [docker push](https://docs.docker.com/engine/reference/commandline/push/) |
| docker /images/docker  | 查看本地镜像                   | [docker /images/docker](https://docs.docker.com/engine/reference/commandline//images/docker/) |
| docker rmi     | 删除本地镜像                   | [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/) |
| docker run     | 创建并运行容器（不能重复创建） | [docker run](https://docs.docker.com/engine/reference/commandline/run/) |
| docker stop    | 停止指定容器                   | [docker stop](https://docs.docker.com/engine/reference/commandline/stop/) |
| docker start   | 启动指定容器                   | [docker start](https://docs.docker.com/engine/reference/commandline/start/) |
| docker restart | 重新启动容器                   | [docker restart](https://docs.docker.com/engine/reference/commandline/restart/) |
| docker rm      | 删除指定容器                   | [docs.docker.com](https://docs.docker.com/engine/reference/commandline/rm/) |
| docker ps      | 查看容器                       | [docker ps](https://docs.docker.com/engine/reference/commandline/ps/) |
| docker logs    | 查看容器运行日志               | [docker logs](https://docs.docker.com/engine/reference/commandline/logs/) |
| docker exec    | 进入容器                       | [docker exec](https://docs.docker.com/engine/reference/commandline/exec/) |
| docker save    | 保存镜像到本地压缩文件         | [docker save](https://docs.docker.com/engine/reference/commandline/save/) |
| docker load    | 加载本地压缩文件到镜像         | [docker load](https://docs.docker.com/engine/reference/commandline/load/) |
| docker inspect | 查看容器详细信息               | [docker inspect](https://docs.docker.com/engine/reference/commandline/inspect/) |

* 一副图来表示这些命令的关系

<img src="/images/docker/image-20240626135821520.png" alt="image-20240626135821520" style="zoom: 80%;" />

* 默认情况下，每次重启虚拟机我们都需要手动启动`Docker`和`Docker`中的容器。通过命令可以实现开机自启

  ```shell
  # 查看Docker是否开启状态
  systemctl status docker
  
  # Docker开机自启
  systemctl enable docker
  
  # Docker关闭开机自启
  systemctl disable docker
  
  # Docker容器开机自启
  docker update --restart=always [容器名/容器id]
  
  # Docker容器关闭自启
  docker update --restart=no [容器名/容器id]
  ```

#### 实战

> 以`Nginx`为例演示上面命令

1. 去[DockerHub](https://hub.docker.com/)查看`nginx`镜像仓库及相关信息

<img src="/images/docker/image-20240626145011879.png" alt="image-20240626145011879" style="zoom: 50%;" />

2. 拉取`nginx`镜像

   ```shell
   docker pull nginx
   ```

   <img src="/images/docker/image-20240626145202943.png" alt="image-20240626145202943" style="zoom: 80%;" />

3. 查看镜像

   ```shell
   docker /images/docker
   ```

   <img src="/images/docker/image-20240626145250275.png" alt="image-20240626145250275" style="zoom: 80%;" />

4. 创建并允许`Nginx`容器

   ```shell
   docker run -d --name nginx -p 80:80 nginx
   ```

   <img src="/images/docker/image-20240626145557188.png" alt="image-20240626145557188" style="zoom:80%;" />

   * -d：在后台运行容器
   * --name： 为容器分配名称
   * -p：设置端口映射（左边是宿主机端口:右边是容器内端口）

5. 查看运行中容器

   ```shell
   docker ps
   ```

   <img src="/images/docker/image-20240626145857607.png" alt="image-20240626145857607" style="zoom:80%;" />

   * 格式化访问，格式会更加清爽

     ```shell
     docker ps --format "table {{.ID}}\t{{.Image}}\t{{.Ports}}\t{{.Status}}\t{{.Names}}"
     ```

     <img src="/images/docker/image-20240626150038118.png" alt="image-20240626150038118" style="zoom:80%;" />

6. 访问网页，测试`nginx`是否启动成功

   ```http
   http://192.168.58.169:80   # 这里填你自己的虚拟机地址
   ```

   <img src="/images/docker/image-20240626150201543.png" alt="image-20240626150201543" style="zoom: 50%;" />

7. 停止容器

   ```shell
   docker stop nginx
   ```

   <img src="/images/docker/image-20240626150317552.png" alt="image-20240626150317552" style="zoom:80%;" />

8. 查看所有容器

   ```shell
   docker ps -a
   ```

   <img src="/images/docker/image-20240626150417609.png" alt="image-20240626150417609" style="zoom:80%;" />

   * 格式化访问，格式会更加清爽

     ```shell
     docker ps -a --format "table {{.ID}}\t{{.Image}}\t{{.Ports}}\t{{.Status}}\t{{.Names}}"
     ```

     <img src="/images/docker/image-20240626150511142.png" alt="image-20240626150511142" style="zoom:80%;" />

9. 再次启动`nginx`容器

   ```shell
   docker start nginx
   ```

   <img src="/images/docker/image-20240626150552526.png" alt="image-20240626150552526" style="zoom:80%;" />

10. 再次查看容器

    ```shell
    docker ps --format "table {{.ID}}\t{{.Image}}\t{{.Ports}}\t{{.Status}}\t{{.Names}}"
    ```

    <img src="/images/docker/image-20240626150644716.png" alt="image-20240626150644716" style="zoom:80%;" />

11. 查看容器详细信息

    ```shell
    docker inspect nginx
    ```

    <img src="/images/docker/image-20240626150820507.png" alt="image-20240626150820507" style="zoom:80%;" />

12. 进入容器,查看容器内目录

    ```shell
    docker exec -it nginx bash
    ```

    <img src="/images/docker/image-20240626150916686.png" alt="image-20240626150916686" style="zoom: 80%;" />

    * 或者，可以进入`MySQL`

      ```shell
      docker exec -it mysql mysql -uroot -p
      ```

13. 进入容器后，退出容器

    ```shell
    exit
    ```

    <img src="/images/docker/image-20240626151131661.png" alt="image-20240626151131661" style="zoom:80%;" />

14. 删除容器

    ```shell
    docker rm nginx
    ```

    <img src="/images/docker/image-20240626151217904.png" alt="image-20240626151217904" style="zoom:80%;" />

    * 发现无法删除，因为容器运行中，强制删除容器（或者你手动停止这个容器，在进行删除）

      ```shell
      docker rm -f nginx
      ```



#### 命令别名

> 给常用`Docker`命令起别名，方便我们访问

```shell
vi /root/.bashrc
```

此时会进入到文件里面，显示的内容如下：

<img src="/images/docker/image-20240626151641488.png" alt="image-20240626151641488" style="zoom:80%;" />

* 这里涉及到一些vi命令的知识~自行百度~

* 简单说下：进入到文件后 按 `i` 就进入了`insert`插入模式，此时就可以修改内容，编辑好了内容，按一下`Esc`键，退出插入模式，此时按 `Shift + ; `键，然后输入`wq`（保存的意思），按回车键即可！

  ![image-20240626151902437](/images/docker/image-20240626151902437.png)

* 最后，执行命令使别名生效

  ```shell
  source /root/.bashrc
  ```

  <img src="/images/docker/image-20240626152011634.png" alt="image-20240626152011634" style="zoom:80%;" />

  * 可以发现，`dis`、`dps`命令都可以正常使用了

### 数据卷

> 容器是隔离环境，容器内程序的文件、配置、运行时产生的容器都在容器内部，我们要读写容器内的文件非常不方便
>
> - 如果要升级`MySQL`版本，需要销毁旧容器，那么数据岂不是跟着被销毁了？
> - `MySQL`、`Nginx`容器运行后，如果我要修改其中的某些配置该怎么办？
> - 我想要让`Nginx`代理我的静态资源怎么办？
>
> 因此，容器提供程序的运行环境，但是**程序运行产生的数据、程序运行依赖的配置都应该与容器耦**。

数据卷就是为了解决上面问题而出现的

#### 什么是数据卷？

> **数据卷（volume）**是一个虚拟目录，是**容器内目录**与**宿主机目录**之间映射的桥梁。

拿`Nginx`来说，有两个关键的目录

* `html`：放置一些静态资源
* `conf`：放置配置文件

要让`Nginx`代理我们的静态资源，最好是放到`html`目录；如果我们要修改`Nginx`的配置，最好是找到`conf`下的`nginx.conf`文件，此时，容器运行的`Nginx`所有的文件都在容器内部，所以我们必须利用**数据卷将两个目录与宿主机目录关联**，从而方便我们操作（利用数据卷可以实现宿主机目录和容器内目录之间的双向映射）

<img src="/images/docker/image-20240626202316931.png" alt="image-20240626202316931" style="zoom:80%;" />

上图中

- 我们创建了两个数据卷：`conf`、`html`
- `Nginx`容器内部的`conf`目录和`html`目录分别与两个数据卷关联
- 而数据卷`conf`和`html`分别指向了宿主机的`/var/lib/docker/volumes/conf/_data`目录和`/var/lib/docker/volumes/html/_data`目录

> 这样一来，容器内的`conf`和`html`目录就 与宿主机的`conf`和`html`目录关联起来，我们称为**挂载**。
>
> 此时，我们操作宿主机的`/var/lib/docker/volumes/html/_data`就是在操作容器内的`/usr/share/nginx/html/_data`目录。
>
> 只要我们将静态资源放入宿主机对应目录，就可以被`Nginx`代理了。



> **小提示**：
>
> `/var/lib/docker/volumes`这个目录就是默认的存放所有容器数据卷的目录，其下再根据数据卷名称创建新目录，格式为`/数据卷名/_data`。
>
> **为什么不让容器目录直接指向宿主机目录呢**？
>
> - 因为直接指向宿主机目录就与宿主机强耦合了，如果切换了环境，宿主机目录就可能发生改变了。由于容器一旦创建，目录挂载就无法修改，这样容器就无法正常工作了。
> - 但是容器指向数据卷，一个逻辑名称，而数据卷再指向宿主机目录，就不存在强耦合。如果宿主机目录发生改变，只要改变数据卷与宿主机目录之间的映射关系即可。
>
> 不过，我们通过由于数据卷目录比较深，不好寻找，通常我们也**允许让容器直接与宿主机目录挂载而不使用数据卷**，具体参考`挂载本地目录或文件`章节笔记

#### 创建数据卷的时机

> 容器与数据卷的挂载要在**创建容器时配置**，对于创建好的容器，是不能设置数据卷的。而且**创建容器的过程中，数据卷会自动创建**

#### 数据库命令

| **命令**              | **说明**             | **文档地址**                                                 |
| :-------------------- | :------------------- | :----------------------------------------------------------- |
| docker volume create  | 创建数据卷           | [docker volume create](https://docs.docker.com/engine/reference/commandline/volume_create/) |
| docker volume ls      | 查看所有数据卷       | [docs.docker.com](https://docs.docker.com/engine/reference/commandline/volume_ls/) |
| docker volume rm      | 删除指定数据卷       | [docs.docker.com](https://docs.docker.com/engine/reference/commandline/volume_prune/) |
| docker volume inspect | 查看某个数据卷的详情 | [docs.docker.com](https://docs.docker.com/engine/reference/commandline/volume_inspect/) |
| docker volume prune   | 清除数据卷           | [docker volume prune](https://docs.docker.com/engine/reference/commandline/volume_prune/) |

##### 实战

* `nginx`的`html`目录挂载

1. 首先创建容器并指定数据卷，注意通过 `-v ` 参数来指定数据卷 （-v 数据卷名:容器内目录）

   ```shell
   docker run -d --name nginx -p 80:80 -v html:/usr/share/nginx/html nginx
   ```

   <img src="/images/docker/image-20240626203344921.png" alt="image-20240626203344921" style="zoom:80%;" />



2. 查看数据卷

   ```shell
   docker volume ls
   ```

   

   <img src="/images/docker/image-20240626203838348.png" alt="image-20240626203838348" style="zoom:80%;" />

3. 查看数据卷详情

   ```shell
   docker volume inspect html
   ```

   <img src="/images/docker/image-20240626204014027.png" alt="image-20240626204014027" style="zoom:80%;" />

4. 查看`/var/lib/docker/volumes/html/_data`目录

   ```shell
   ll /var/lib/docker/volumes/html/_data
   ```

   <img src="/images/docker/image-20240626204056197.png" alt="image-20240626204056197" style="zoom:80%;" />

   * 可以看到和`nginx`容器里面的`html`目录内容一样

   ![image-20240626210040997](/images/docker/image-20240626210040997.png)

5. 进入到宿主机对应的目录，并随意修改`index.html`内容

   ```shell
   cd /var/lib/docker/volumes/html/_data
   vi index.html
   ```

   <img src="/images/docker/image-20240626210647262.png" alt="image-20240626210647262" style="zoom:80%;" />

   <img src="/images/docker/image-20240626210832622.png" alt="image-20240626210832622" style="zoom:80%;" />

6. 访问`nginx`，查看效果

   <img src="/images/docker/image-20240626210957257.png" alt="image-20240626210957257" style="zoom: 67%;" />

7. 我们刚刚修改了宿主机里面的内容，我们现在进入容器内部，查看`/usr/share/nginx/html`目录内的文件是否变化

   <img src="/images/docker/image-20240626211243717.png" alt="image-20240626211243717" style="zoom: 67%;" />

   * **可以发现，我们刚刚修改了宿主机的文件内容，容器内部的文件内容也发生了变化，测试可得：是同步的。**



#### `MySQL`的匿名数据卷

1. 查看`MySQL`容器详细信息 

   ```shell
   docker inspect mysql
   ```

   关注两部分内容，第一是`.Config.Volumes`部分

   ```shell
   {
     "Config": {
       // ... 略
       "Volumes": {
         "/var/lib/mysql": {}
       }
       // ... 略
     }
   }
   ```

   可以发现这个容器声明了一个本地目录，需要挂载数据卷，但是**数据卷未定义**。这就是匿名卷。再看结果中的`.Mounts`部分：

   ```shell
   {
     "Mounts": [
               {
                   "Type": "volume",
                   "Name": "e90f6c7f57224c4c9d43dfddd0e7ff44dc33001ab476e718129441d3aef11960",
                   "Source": "/var/lib/docker/volumes/e90f6c7f57224c4c9d43dfddd0e7ff44dc33001ab476e718129441d3aef11960/_data",
                   "Destination": "/var/lib/mysql",
                   "Driver": "local",
                   "Mode": "",
                   "RW": true,
                   "Propagation": ""
               }
           ]
   }
   ```

   > - Name：数据卷名称。由于定义容器未设置容器名，这里的就是匿名卷自动生成的名字，一串hash值。
   > - Source：宿主机目录
   > - Destination : 容器内的目录

上述配置是将容器内的`/var/lib/mysql`这个目录，与数据卷`e90f6c7f57224c4c9d43dfddd0e7ff44dc33001ab476e718129441d3aef11960`挂载。于是在宿主机中就有了`/var/lib/docker/volumes/e90f6c7f57224c4c9d43dfddd0e7ff44dc33001ab476e718129441d3aef11960/_data`这个目录。这就是匿名数据卷对应的目录，其使用方式与普通数据卷没有差别。

![image-20240626212116627](/images/docker/image-20240626212116627.png)

2. 可以查看该目录下的`MySQL`的`data`文件

   ```shell
   ls -l /var/lib/docker/volumes/e90f6c7f57224c4c9d43dfddd0e7ff44dc33001ab476e718129441d3aef11960/_data
   ```

   <img src="/images/docker/image-20240626212248889.png" alt="image-20240626212248889" style="zoom: 50%;" />

#### 挂载本地目录或文件

> 数据卷的**目录结构较深**，如果我们去操作数据卷目录会不太方便。在很多情况下，我们会直接将**容器目录与宿主机指定目录挂载**

```shell
# 挂载本地目录
-v 本地目录:容器内目录
# 挂载本地文件
-v 本地文件:容器内文件
```

**注意**：本地目录或文件必须以 `/` 或 `./`开头，如果直接以名字开头，会被识别为数据卷名而非本地目录名

```Bash
-v mysql:/var/lib/mysql # 会被识别为一个数据卷叫mysql，运行时会自动创建这个数据卷
-v ./mysql:/var/lib/mysql # 会被识别为当前目录下的mysql目录，运行时如果不存在会创建目录
```

##### 实战

* 删除并重新创建`mysql`容器，并完成本地目录挂载
  - 挂载`/root/mysql/data`到容器内的`/var/lib/mysql`目录
  - 挂载`/root/mysql/init`到容器内的`/docker-entrypoint-initdb.d`目录（初始化的`SQL`脚本目录）
  - 挂载`/root/mysql/conf`到容器内的`/etc/mysql/conf.d`目录（这个是`MySQL`配置文件目录）



* 资料中已经准备好了`mysql`的`init`目录和`conf`目录

<img src="/images/docker/image-20240626213109660.png" alt="image-20240626213109660" style="zoom: 80%;" />

<img src="/images/docker/image-20240626213221897.png" alt="image-20240626213221897" style="zoom: 80%;" />

* `hm.cnf`主要是配置了`MySQL`的默认编码，改为`utf8mb4`；而`hmall.sql`则是黑马商城项目的初始化`SQL`脚本

* 将整个`mysql`目录上传至虚拟机的`/root`目录下

  <img src="/images/docker/image-20240626213607069.png" alt="image-20240626213607069" style="zoom:50%;" />

1. 删除原来的`MySQL`容器

   ```shell
   docker rm -f mysql
   ```

   <img src="/images/docker/image-20240626213807114.png" alt="image-20240626213807114" style="zoom: 50%;" />

2. 创建并运行新`mysql`容器，挂载本地目录

   ```shell
   docker run -d \
     --name mysql \
     -p 3306:3306 \
     -e TZ=Asia/Shanghai \
     -e MYSQL_ROOT_PASSWORD=123 \
     -v ./mysql/data:/var/lib/mysql \
     -v ./mysql/conf:/etc/mysql/conf.d \
     -v ./mysql/init:/docker-entrypoint-initdb.d \
     mysql
   ```

   <img src="/images/docker/image-20240626213941002.png" alt="image-20240626213941002" style="zoom:50%;" />

3. 查看`root`目录，可以发现`~/mysql/data`目录已经自动创建好了

   ```shell
   ls -l mysql
   ```

   <img src="/images/docker/image-20240626214033177.png" alt="image-20240626214033177" style="zoom:80%;" />

4. 查看`data`目录，会发现里面有大量数据库数据，说明数据库完成了初始化

   ```shell
   cd mysql
   ls -l data
   ```

   <img src="/images/docker/image-20240626214227875.png" alt="image-20240626214227875" style="zoom:50%;" />

   5. 进入`MySQL`

      ```shell
      docker exec -it mysql mysql -uroot -p123
      ```

      <img src="/images/docker/image-20240626214342442.png" alt="image-20240626214342442" style="zoom:50%;" />

   6. 查看编码表

      ```sql
      show variables like "%char%";
      ```

      <img src="/images/docker/image-20240626214409893.png" alt="image-20240626214409893" style="zoom: 50%;" />

   7. 查看数据库

      ```sql
      show databases;
      ```

      <img src="/images/docker/image-20240626214457227.png" alt="image-20240626214457227" style="zoom:50%;" />

   8. 切换到`hmall`数据库

      ```sql
      use hmall;
      ```

      <img src="/images/docker/image-20240626214532947.png" alt="image-20240626214532947" style="zoom:50%;" />

   9. 查看表

      ```sql
      show tables;
      ```

      <img src="/images/docker/image-20240626214601202.png" alt="image-20240626214601202" style="zoom:50%;" />

   10. 查看`address`表数据

       ```sql
       select * from address
       ```

       <img src="/images/docker/image-20240626214706111.png" alt="image-20240626214706111" style="zoom:50%;" />

### 镜像

* 一直在使用别人准备好的镜像，那如果我要部署一个`Java`项目，把它打包为一个镜像该怎么做呢？？？

#### 结构

> 镜像之所以能让我们快速跨操作系统部署应用而忽略其运行环境、配置，就是因为镜像中包含了程序运行需要的系统函数库、环境、配置、依赖。
>
> 因此，自定义镜像本质就是依次准备好程序运行的基础环境、依赖、应用本身、运行配置等文件，并且打包而成
>
> 举个例子，我们要从0部署一个Java应用，大概流程是这样：
>
> - 准备一个`linux`服务（`CentOS`或者`Ubuntu`均可）
> - 安装并配置`JDK`
> - 上传Jar包
> - 运行jar包
>
> 那因此，我们打包镜像也是分成这么几步：
>
> - 准备Linux运行环境（`java`项目并不需要完整的操作系统，仅仅是基础运行环境即可）
> - 安装并配置`JDK`
> - 拷贝jar包
> - 配置启动脚本
>
> 上述步骤中的每一次操作其实都是在生产一些文件（系统运行环境、函数库、配置最终都是磁盘文件），所以**镜像就是一堆文件的集合**。
>
> 镜像文件不是随意堆放的，而是按照操作的步骤分层叠加而成，每一层形成的文件都会单独打包并标记一个唯一id，称为**Layer**（**层**）。这样，如果我们构建时用到的某些层其他人已经制作过，就可以直接拷贝使用这些层，而不用重复制作。

第一步中需要的`Linux`运行环境，通用性就很强，所以`Docker`官方就制作了这样的只包含`Linux`运行环境的镜像。我们在制作`java`镜像时，就无需重复制作，直接使用Docker官方提供的`CentOS`或`Ubuntu`镜像作为基础镜像。然后再搭建其它层即可，这样逐层搭建，最终整个`Java`项目的镜像结构如图所示

<img src="/images/docker/image-20240626215155460.png" alt="image-20240626215155460" style="zoom:80%;" />

#### `Dockerfile`

> 由于制作镜像的过程中，需要逐层处理和打包，比较复杂，所以`Docker`就提供了自动打包镜像的功能。我们只需要将打包的过程，每一层要做的事情用固定的语法写下来，交给`Docker`去执行即可，而这种记录镜像结构的文件就称为**`Dockerfile`**
>
> * https://docs.docker.com/engine/reference/builder/ ：对应语法参考文档，其中的语法比较多，比较常用的有：
>
>   | **指令**         | **说明**                                     | **示例**                      |
>   | :--------------- | :------------------------------------------- | :---------------------------- |
>   | **`FROM`**       | 指定基础镜像                                 | `FROM centos:6`               |
>   | **`ENV`**        | 设置环境变量，可在后面指令使用               | `ENV key value`               |
>   | **`COPY`**       | 拷贝本地文件到镜像的指定目录                 | `COPY ./xx.jar /tmp/app.jar`  |
>   | **`RUN`**        | 执行Linux的shell命令，一般是安装过程的命令   | `RUN yum install gcc`         |
>   | **`EXPOSE`**     | 指定容器运行时监听的端口，是给镜像使用者看的 | `EXPOSE 8080`                 |
>   | **`ENTRYPOINT`** | 镜像中应用的启动命令，容器运行时调用         | `ENTRYPOINT java -jar xx.jar` |

* 例如，要基于`Ubuntu`镜像来构建一个Java应用，其`Dockerfile`内容如下

  ```dockerfile
  # 指定基础镜像
  FROM ubuntu:16.04
  # 配置环境变量，JDK的安装目录、容器内时区
  ENV JAVA_DIR=/usr/local
  ENV TZ=Asia/Shanghai
  # 拷贝jdk和java项目的包
  COPY ./jdk8.tar.gz $JAVA_DIR/
  COPY ./docker-demo.jar /tmp/app.jar
  # 设定时区
  RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
  # 安装JDK
  RUN cd $JAVA_DIR \
   && tar -xf ./jdk8.tar.gz \
   && mv ./jdk1.8.0_144 ./java8
  # 配置环境变量
  ENV JAVA_HOME=$JAVA_DIR/java8
  ENV PATH=$PATH:$JAVA_HOME/bin
  # 指定项目监听的端口
  EXPOSE 8080
  # 入口，java项目的启动命令
  ENTRYPOINT ["java", "-jar", "/app.jar"]
  ```

> 思考一下？
>
> * 以后会有很多很多`java`项目需要打包为镜像，他们都需要`Linux`系统环境、`JDK`环境这两层，只有上面的3层不同（因为jar包不同）。
>
> * 如果每次制作`java`镜像都重复制作前两层镜像，是不是很麻烦

* 所以，就有人提供了基础的系统加`JDK`环境，我们在此基础上制作`java`镜像，就可以省去`JDK`的配置了

  ```dockerfile
  # 基础镜像
  FROM openjdk:11.0-jre-buster
  # 设定时区
  ENV TZ=Asia/Shanghai
  RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
  # 拷贝jar包
  COPY docker-demo.jar /app.jar
  # 入口
  ENTRYPOINT ["java", "-jar", "/app.jar"]
  ```

#### 构建镜像

> 当`Dockerfile`文件写好以后，就可以利用命令来构建镜像了

* 首先，我们把一个`boot`项目打好的`jar`包和对应的`Dockerfile`文件拷贝到虚拟机的`/root/demo`目录下

  `Dockerfile`内容如下：

  ```dockerfile
  FROM openjdk:11.0-jre-buster
  ENV TZ=Asia/Shanghai
  RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
  COPY hm-service.jar /app.jar
  ENTRYPOINT ["java", "-jar", "/app.jar"]
  ```

* 查看/root/demo下文件

  <img src="/images/docker/image-20240626220303842.png" alt="image-20240626220303842" style="zoom:50%;" />

* 执行命令，构建镜像

  ```shell
  # 开始构建
  docker build -t docker-demo:1.0 .
  ```

  命令说明：

  - `docker build `: 就是构建一个docker镜像
  - `-t docker-demo:1.0` ：`-t` 镜像名称:版本号
  - `.` : 最后的点是指构建时`Dockerfile`所在路径，由于我们进入了`demo`目录，所以指定的是`.`代表当前目录，也可以直接指定`Dockerfile`目录：
    - ```Bash
      # 直接指定Dockerfile目录
      docker build -t docker-demo:1.0 /root/demo
      ```

<img src="/images/docker/image-20240626221219650.png" alt="image-20240626221219650" style="zoom: 50%;" />

* 查看镜像列表

  ```shell
  docker /images/docker
  ```

  

  <img src="/images/docker/image-20240626221309308.png" alt="image-20240626221309308" style="zoom:50%;" />

* 运行该镜像

  ```shell
  # 1.创建并运行容器
  docker run -d --name dd -p 8082:8082 docker-demo:1.0
  # 2.查看容器 (我配置了别名，所以dps = docker ps)
  dps  
  # 3.浏览器访问
  localhost:8080/hello/count
  # 结果：
  <h5>欢迎访问黑马商城, 这是您第1次访问<h5>
  ```

  <img src="/images/docker/image-20240626221644978.png" alt="image-20240626221644978" style="zoom:50%;" />

  * 浏览器访问：`http://您的虚拟机ip地址:您的端口/您的接口路径`

    ![image-20240626222015693](/images/docker/image-20240626222015693.png)

### 网络

> `Java`项目往往需要访问其它各种中间件，例如`MySQL`、`Redis`等。现在，我们的容器之间能否互相访问呢？？？

*  查看下`MySQL`容器的详细信息，重点关注其中的网络`IP`地址（**每个容器都有一个`ip`地址**）

1. 基本命令查找，寻找`Networks.bridge.IPAddress`属性

   ```shell
   docker inspect mysql
   ```

   <img src="/images/docker/image-20240627092908048.png" alt="image-20240627092908048" style="zoom:80%;" />

   * 也可以使用`format`过滤结果

     ```shell
     docker inspect --format='{{range .NetworkSettings.Networks}}{{println .IPAddress}}{{end}}' mysql
     ```

     <img src="/images/docker/image-20240627092954276.png" alt="image-20240627092954276" style="zoom:80%;" />

2. 然后通过命令进入`dd`容器

   ```shell
   docker exec -it dd bash
   ```

   <img src="/images/docker/image-20240627093131965.png" alt="image-20240627093131965" style="zoom: 67%;" />

3. 在容器内，通过`ping`命令测试网络，可以发现互联

   ```shell
   ping 172.17.0.3
   ```

   <img src="/images/docker/image-20240627093504027.png" alt="image-20240627093504027" style="zoom:80%;" />

   * 注意：`mysql`容器必须在启动中状态，不然`ping`的时候会报错如下：

     <img src="/images/docker/image-20240627093531702.png" alt="image-20240627093531702" style="zoom:80%;" />



> 可能发生的问题？
>
> * 容器的网络`IP`其实是一个虚拟的`IP`，其值并不固定与某一个容器绑定，如果我们在开发时写死某个`IP`，而在部署时很可能`MySQL`容器的`IP`会发生变化，连接会失败

所以，`docker`的网络功能的出现就可以解决这个问题，常见命令有

| **命令**                  | **说明**                 | **文档地址**                                                 |
| :------------------------ | :----------------------- | :----------------------------------------------------------- |
| docker network create     | 创建一个网络             | [docker network create](https://docs.docker.com/engine/reference/commandline/network_create/) |
| docker network ls         | 查看所有网络             | [docs.docker.com](https://docs.docker.com/engine/reference/commandline/network_ls/) |
| docker network rm         | 删除指定网络             | [docs.docker.com](https://docs.docker.com/engine/reference/commandline/network_rm/) |
| docker network prune      | 清除未使用的网络         | [docs.docker.com](https://docs.docker.com/engine/reference/commandline/network_prune/) |
| docker network connect    | 使指定容器连接加入某网络 | [docs.docker.com](https://docs.docker.com/engine/reference/commandline/network_connect/) |
| docker network disconnect | 使指定容器连接离开某网络 | [docker network disconnect](https://docs.docker.com/engine/reference/commandline/network_disconnect/) |
| docker network inspect    | 查看网络详细信息         | [docker network inspect](https://docs.docker.com/engine/reference/commandline/network_inspect/) |

#### 自定义网络

1. 首先通过命令创建一个网络

   ```shell
   docker network create yjx
   ```

   <img src="/images/docker/image-20240627093844437.png" alt="image-20240627093844437" style="zoom:80%;" />

2. 然后查看网络

   ```shell
   docker network ls
   ```

   <img src="/images/docker/image-20240627093916616.png" alt="image-20240627093916616" style="zoom:80%;" />

   * 其中，除了`yjx`以外，其它都是默认的网络

3. 让`dd`和`mysql`容器都加入该网络

   * 加入网络时可以通过`--alias`给容器起别名，这样该网络内的其它容器可以用**别名互相访问**！

   ```shell
   # 指定别名为db，另外每一个容器都有一个别名是容器名
   docker network connect yjx mysql --alias db
   # dd容器，也就是我们的java项目
   docker network connect yjx dd
   ```

   <img src="/images/docker/image-20240627094215890.png" alt="image-20240627094215890" style="zoom:80%;" />

4. 进入`dd`容器，尝试利用别名访问`db`

   ```shell
   docker exec -it dd bash  # 进入容器
   ping db  # 用db别名访问
   ping mysql  # 用容器名访问
   ```

   <img src="/images/docker/image-20240627094314579.png" alt="image-20240627094314579" style="zoom:80%;" />

   <img src="/images/docker/image-20240627094409929.png" alt="image-20240627094409929" style="zoom:80%;" />



> **现在无需记住`IP`地址也可以实现容器互联了**
>
> **总结**
>
> - 在自定义网络中，可以给容器起多个别名，默认的别名是容器名本身
> - 在同一个自定义网络中的容器，可以通过别名互相访问



## 项目部署

### 部署Java项目

1. 将您的`Dockerfile`和`jar`包一起上传到虚拟机的`root`目录

   <img src="/images/docker/image-20240627095137630.png" alt="image-20240627095137630" style="zoom:80%;" />

2. 构建项目镜像，不指定`tag`，则默认为`latest`

   ```shell
   docker build -t yjxx .
   ```

   <img src="/images/docker/image-20240627095351612.png" alt="image-20240627095351612" style="zoom:80%;" />

3. 查看镜像

   ```shell
   docker /images/docker
   ```

   <img src="/images/docker/image-20240627095419810.png" alt="image-20240627095419810" style="zoom:80%;" />

4. 创建并运行容器，并通过`--network`将其加入`yjx`网络，这样才能通过容器名访问`mysql`

   * 我项目的配置文件

![image-20240627095634395](/images/docker/image-20240627095634395.png)

```shell
docker run -d --name test --network yjx -p 8082:8082 yjxx
```

<img src="/images/docker/image-20240627095722518.png" alt="image-20240627095722518" style="zoom:80%;" />

5. 测试，通过浏览器访问：`http://你的虚拟机地址:你的端口/你的接口地址`

<img src="/images/docker/image-20240627095824674.png" alt="image-20240627095824674" style="zoom:50%;" />



### 部署前端

> 基于`nginx`部署

* `nginx`内容如下

<img src="/images/docker/image-20240627100023059.png" alt="image-20240627100023059" style="zoom: 80%;" />

* `nginx/html`内容如下，有两个前端项目

<img src="/images/docker/image-20240627100148067.png" alt="image-20240627100148067" style="zoom: 67%;" />

* `nginx.conf`内容如下

  <img src="/images/docker/image-20240627103823987.png" alt="image-20240627103823987" style="zoom:80%;" />



1. 把整个`nginx`目录上传到虚拟机的`/root`目录下

   <img src="/images/docker/image-20240627102621252.png" alt="image-20240627102621252" style="zoom:80%;" />

2. 创建`nginx`容器并完成两个挂载

   - 把`/root/nginx/nginx.conf`挂载到`/etc/nginx/nginx.conf`
   - 把`/root/nginx/html`挂载到`/usr/share/nginx/html`

   > 如下根据你自己的项目代理端口情况指定~

   * 需要让`nginx`同时代理`hmall-portal`和`hmall-admin`两套前端资源，因此我们需要暴露两个端口：
     * 18080：对应`hmall-portal`
     * 18081：对应`hmall-admin`

   ```shell
   docker run -d \
     --name nginx \
     -p 18080:18080 \
     -p 18081:18081 \
     -v /root/nginx/html:/usr/share/nginx/html \
     -v /root/nginx/nginx.conf:/etc/nginx/nginx.conf \
     --network yjx \
     nginx
   ```

   <img src="/images/docker/image-20240627103154602.png" alt="image-20240627103154602" style="zoom:80%;" />

* 看下`nginx`容器有没有启动

  <img src="/images/docker/image-20240627103449942.png" alt="image-20240627103449942" style="zoom:80%;" />

* 浏览器访问你前端项目地址测试：`http://你的虚拟机ip:您暴露的端口`

  <img src="/images/docker/image-20240627103952450.png" alt="image-20240627103952450" style="zoom:80%;" />

### `DockerCompose`

> 部署一个简单的`java`项目，其中包含`3`个容器
>
> - `MySQL`
> - `Nginx`
> - `Java`项目
>
> 稍微复杂的项目，其中还会有各种各样的其它中间件，需要部署的东西远不止3个。如果还像之前那样手动的逐一部署，就太麻烦了

* Docker Compose就可以帮助我们实现**多个相互关联的Docker容器的快速部署**
* 允许用户通过一个单独的 `docker-compose.yml `模板文件（`YAML` 格式）来定义一组相关联的应用容器

#### 基本语法

* `docker-compose.yml`文件的基本语法可以参考官方文档
  * https://docs.docker.com/compose/compose-file/legacy-versions/



> `docker-compose`文件中可以定义多个相互关联的应用容器，每一个应用容器被称为一个**服务**（service）
>
> * 用`docker run`部署`MySQL`的命令如下
>
>   ```shell
>   docker run -d \
>     --name mysql \
>     -p 3306:3306 \
>     -e TZ=Asia/Shanghai \
>     -e MYSQL_ROOT_PASSWORD=123 \
>     -v ./mysql/data:/var/lib/mysql \
>     -v ./mysql/conf:/etc/mysql/conf.d \
>     -v ./mysql/init:/docker-entrypoint-initdb.d \
>     --network yjx
>     mysql
>   ```
>
> * 用`docker-compose.yml`文件来定义，就是这样：
>
>   ```shell
>   services:
>     mysql:
>       image: mysql
>       container_name: mysql
>       ports:
>         - "3306:3306"
>       environment:
>         TZ: Asia/Shanghai
>         MYSQL_ROOT_PASSWORD: 123
>       volumes:
>         - "./mysql/conf:/etc/mysql/conf.d"
>         - "./mysql/data:/var/lib/mysql"
>       networks:
>         - new
>   networks:
>     new:
>       name: yjx
>   ```
>
>   | **docker run 参数** | **docker compose 指令** | **说明**   |
>   | :------------------ | :---------------------- | :--------- |
>   | --name              | container_name          | 容器名称   |
>   | -p                  | ports                   | 端口映射   |
>   | -e                  | environment             | 环境变量   |
>   | -v                  | volumes                 | 数据卷配置 |
>   | --network           | networks                | 网络       |
>
> 跟着上面的例子，把我们对应项目的`docker-compose.yml`定义好
>
> ```yaml
> services:
>   mysql:
>     image: mysql # 使用mysql镜像
>     container_name: mysql # 指定容器名称 容器名为mysql
>     ports:
>       - "3306:3306" # 将主机的3306端口映射到容器的3306端口
>     environment:
>       TZ: Asia/Shanghai # 设置环境变量TZ为Asia/Shanghai
>       MYSQL_ROOT_PASSWORD: 123 # 设置环境变量MYSQL_ROOT_PASSWORD为123
>     volumes:
>       - "./mysql/conf:/etc/mysql/conf.d" # 将主机的./mysql/conf目录挂载到容器的/etc/mysql/conf.d目录
>       - "./mysql/data:/var/lib/mysql" # 将主机的./mysql/data目录挂载到容器的/var/lib/mysql目录
>       - "./mysql/init:/docker-entrypoint-initdb.d" # 将主机的./mysql/init目录挂载到容器的/docker-entrypoint-initdb.d目录
>     networks:
>       - yjx-net # 加入yjx-net网络
>   yjx:
>     build:
>       context: .  # 使用当前目录的Dockerfile构建镜像
>       dockerfile: Dockerfile  # 使用名为Dockerfile的文件来构建Docker镜像
>     container_name: yjx # 容器名为yjx
>     ports:
>       - "8082:8082"  # 将主机的8082端口映射到容器的8082端口
>     networks:
>       - yjx-net # 加入yjx-net网络
>     depends_on:
>       - mysql  # 依赖mysql服务
>   nginx:
>     image: nginx # 使用nginx镜像
>     container_name: nginx # 容器名为nginx
>     ports:
>       - "18080:18080" # 将主机的18080端口映射到容器的18080端口
>       - "18081:18081" # 将主机的18081端口映射到容器的18081端口
>     volumes:
>       - "./nginx/nginx.conf:/etc/nginx/nginx.conf" # 将主机的./nginx/nginx.conf文件挂载到容器的/etc/nginx/nginx.conf文件
>       - "./nginx/html:/usr/share/nginx/html" # 将主机的./nginx/html目录挂载到容器的/usr/share/nginx/html目录
>     depends_on:
>       - yjx # 依赖yjx服务
>     networks:
>       - yjx-net # 加入yjx-net网络
> networks:
>   yjx-net:
>     name: yjx # 指定网络的名称
> ```
>
> 

#### 基础命令

* 编写好`docker-compose.yml`	文件，就可以部署项目了，常见的命令看文档：https://docs.docker.com/compose/reference/

基本语法如下：

```dockerfile
docker compose [OPTIONS] [COMMAND]
```

* `OPTIONS`和`COMMAND`都是可选参数，比较常见的有：

  <img src="/images/docker/image-20240627104334256.png" alt="image-20240627104334256" style="zoom:80%;" />

##### 实战

1. 进入`root`目录

   ```shell
   cd /root
   ```

   <img src="/images/docker/image-20240627105902837.png" alt="image-20240627105902837" style="zoom:80%;" />

2. 把编写好的`docker-compose.yml`放入`root`目录下

   <img src="/images/docker/image-20240627110255382.png" alt="image-20240627110255382" style="zoom:80%;" />

3. 删除之前的旧容器

   ```shell
   docker rm -f $(docker ps -qa)   # 删除所有正在运行和停止的Docker容器
   ```

   <img src="/images/docker/image-20240627110016000.png" alt="image-20240627110016000" style="zoom:80%;" />

4. 记得把之前的网络也删掉

   ```shell
   docker network rm 网络名称
   ```

   <img src="/images/docker/image-20240627110851977.png" alt="image-20240627110851977" style="zoom:80%;" />

5. 启动所有, `-d` 参数是后台启动

   ```shell
   docker compose up -d
   ```

   <img src="/images/docker/image-20240627112031509.png" alt="image-20240627112031509" style="zoom:67%;" />

6. 查看镜像

   ```shell
   docker compose /images/docker
   ```

   <img src="/images/docker/image-20240627112120386.png" alt="image-20240627112120386" style="zoom: 80%;" />

7. 查看容器

   ```shell
   docker compose ps
   ```

   <img src="/images/docker/image-20240627112157219.png" alt="image-20240627112157219" style="zoom:80%;" />

8. 测试，打开浏览器访问：`http://您的虚拟机ip:您的端口`

   * 后端服务访问成功

     <img src="/images/docker/image-20240627112306555.png" alt="image-20240627112306555" style="zoom:80%;" />

   * 前端项目访问成功

     <img src="/images/docker/image-20240627112334123.png" alt="image-20240627112334123" style="zoom: 67%;" />