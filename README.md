## Docker

#### 参考方向
* Java  - Spring / Spring Boot / Spring MVC / Spring Data / Mybatis / Dubbo / RocketMQ ...
* Go - Swarm / Compose / Machine / mesos / K8S ... (CI/CD - jenkinds整合)

#### Docker基础篇 (2020-01-13)
1. 理念: 在任何地方构建, 发布并运行任何应用.  

2. 解释: 解决了运行环境和配置问题软件容器, 方便做持续集成并有助于争议发布的容器虚拟化技术.  

3. 三要素:  
     - 仓库: 集中存放镜像的场所.
       * 仓库注册服务器 ( `Registry` ) 和仓库 ( `Repository` ) 是有区别的.  
        - 仓库注册服务器上可以存放多个仓库
        - 仓库中存放多个镜像, 每个镜像有不同的标签 ( `Tag` ).
        - 仓库分为公开仓库 ( `Public` ) 和私有仓库 ( `Private` ) 两种形式.
          * 最大的公开仓库是 [Docker Hub](https://hub.docker.com).
          - 国内比较流行的仓库
            * 阿里云 (https://xxx.mirror.aliyuncs.com) (可存放私有镜像, 需自行[注册](https://promotion.aliyun.com/ntms/act/kubernetes.html))
            * 网易云 (http://hub-mirror.c.163.com)
        - 私有仓库需自行搭建, 目前比较流行的私有仓库是[Harbor](https://github.com/goharbor/harbor).
     - 镜像: 一个只读的模板. 可以用来创建Docker容器. 一个镜像可以创建很多容器.
     - 容器: 容器是镜像创建的运行实例.

4. 通俗理解:  

    <div style="display: flex; width: 100%">
        <div style="width: 15%; margin-left: 20px">
            <img src="assets/docker-logo.png" alt="Docker logo">
        </div>
        <div style="width: 85%; margin-left: 20px">
            <p>从 <code>Java</code> 和 logo 来看 <code>Docker</code> - 一条蓝色的鲸鱼, 背着很多集装箱.</p>
            <table>
                <thead>
                    <tr>
                        <th style="text-align:center">Java</th>
                        <th style="text-align:center">Docker</th>
                        <th style="text-align:center">logo</th>
                        <th style="text-align:center">解释</th>
                    </tr>
                </thead>
                <tbody>
                    <tr>
                        <td style="text-align:center">项目</td>
                        <td style="text-align:center">仓库</td>
                        <td style="text-align:center">地球<br>(海洋存方的地方, logo中未显示)</td>
                        <td style="text-align:center">镜像存方的位置<br>(上文提到的公开/私有仓库)</td>
                    </tr>
                    <tr>
                        <td style="text-align:center">类</td>
                        <td style="text-align:center">镜像</td>
                        <td style="text-align:center">要运到鲸鱼背上的集装箱</td>
                        <td style="text-align:center">运行的模板</td>
                    </tr>
                    <tr>
                        <td style="text-align:center">对象</td>
                        <td style="text-align:center">容器</td>
                        <td style="text-align:center">鲸鱼背上的集装箱</td>
                        <td style="text-align:center"><code>Docker</code> 实例<br>(来自镜像)</td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>

5. 安装:
   - CentOS: (yum / rpm) 参考[官方教程](https://docs.docker.com/install/linux/docker-ce/centos/)
   - 通用安装:  
     ```
     curl -fsSL https://get.docker.com -o get-docker.sh
     sudo sh get-docker.sh --mirror Aliyun/AzureChinaCloud
     ```

6. 配置国内镜像:  
   ```
   sudo mkdir -p /etc/docker
   sudo tee /etc/docker/daemon.json <<-'EOF'
   { 
       "registry-mirrors": 
           [ 
               "http://hub-mirror.c.163.com", 
               "https://xxx.mirror.aliyuncs.com" 
           ]
   }
   EOF
   sudo systemctl daemon-reload
   sudo systemctl restart docker
   ```

7. `Docker` 为什么比 `VM` 快?
   - `Docker` 有着比 `VM` 更少的抽象层(`Hypervisor`).  
     运行在 `Docker` 上的程序是直接使用宿主机的物理硬件资源,  
     因此在 `CPU` 和 `RAM` 的运行效率上会明显优势.
   - `Docker` 利用的宿主机(`Host`)内核, 而不需要 `GuestOS`.  
     - 当新建一个 `Docker` 容器时, `Docker` 不需要和虚拟机一样重新加载一个操作系统内核.  
       因而避免了引寻, 加载操作系统内核等一些比较费时费资源的过程.
     - 当新建一个 `VM` 时, 需要加载 `GuestOS` 进行硬件虚拟化并初始化 `GuestOS` 的内核,
       在资源分配和调度上耗时较长.
     - `Docker` 和 `VM` 比较
     
        |     -      |      Docker容器       |         虚拟机(VM)         |
        | :--------: | :-------------------: | :------------------------: |
        |  操作系统  |     与宿主机共享      |    宿主机上运行虚拟机OS    |
        |  存储大小  | 镜像小,便于存储于传输 |   镜像庞大(vmdk, vdi等)    |
        |    性能    |  几乎无额外性能损失   | 操作系统额外的CPU,内存消耗 |
        |   移植性   |      轻便, 灵活       | 笨重,与虚拟化技术耦合度高  |
        | 硬件亲和性 |    面向软件开发者     |       面向硬件运维者       |
        |  部署速度  |      快速 (秒级)      |       较慢 (分钟级)        |

8. 常用命令:
   1. 帮助命令
      - docker version
      - docker info
      - docker --help
      
   2. 镜像命令
      - docker images ${sub cmd}
        * 解释: 列出本地镜像 (同一仓库源可以有多个 `TAG`, 代表这个仓库源的不同版本, 可以使用 `REPOSITORY:TAG` 来定义不用的镜像.)
          - REPOSITORY: 镜像仓库源
          - TAG: 镜像标签
          - IMAGE ID: 镜像ID
          - CREATED: 镜像创建时间
          - SIZE: 镜像大小
        * 子命令:
          - -a: 列出本地所有的镜像(含中间映象层)
          - -q: 只显示镜像ID
          - --digests: 显示镜像的摘要(sha256)信息
          - --no-trunc: 显示镜像的完整信息
      - docker search ${sub cmd} ${image name}
        * 解释: 搜索DockerHub上的镜像
          - NAME: 镜像名
          - DESCRIPTION: 镜像描述
          - STARS: 星标数 (等同于 `Github` 的星)
          - OFFICIAL: 是否为官方镜像
          - AUTOMATED: 是否为自动构建的
        * 子命令:
          - -s: 显示大于指定星数的镜像
          - --no-trunc: 显示镜像的完整信息
          - --automated: 只显示自动构建类型的镜像
      - docker pull ${image name}:${tag}
        * 解释: 拉取指定镜像
      - docker rmi ${sub cmd} ${image name / image id}:${tag}
        * 解释: 删除指定镜像
        * 子命令:
          - -f: 强制删除
      
   3. 容器命令
      - docker run ${sub cmd} ${image name / image id} ${cmd} ${arg}
        * 解释: 新建并启动容器
        * 子命令:
          - --name ${container name} : 为容器指定一个名字
          - -d: 后台模式运行容器, 并返回容器ID.
          - -i: 交互模式运行容器, 通常和 `-t` 一起使用.
          - -P: 随机分配短裤
          - -p: 指定端口
            - ip:host port: container port
            - ip::container port
            - host port: container port
            - container port
      - docker ps ${sub cmd}
        * 解释: 同 `Linux`的 `ps`
        * 子命令:
          - -a --all : 显示所有容器(默认只显示正在运行的容器)
          - -l --latest : 最后创建的容器
          - -n --last ${num} : 最后创建的${num}的容器
          - -q : 只显示镜像ID
      - docker start ${container id / container name}
        * 解释: 启动一个容器
      - docker resart
        * 解释: 重启一个容器

<!-- #### Docker高级篇 -->
