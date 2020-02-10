## Docker

### Docker基础篇 (2020-01-13)

>---

##### `Docker` 的概念

1. 理念: 在任何地方构建, 发布并运行任何应用.  

2. 解释: 解决了运行环境和配置问题软件容器, 方便做持续集成并有助于整体发布的容器虚拟化技术.  

3. 三要素:
    - **仓库**: 集中存放镜像的场所.
      - 仓库注册服务器 ( `Registry` ) 和仓库 ( `Repository` ) 是有区别的.  
        - 仓库注册服务器上可以存放多个仓库
        - 仓库中存放多个镜像, 每个镜像有不同的标签 ( `Tag` ).
        - 仓库分为公开仓库 ( `Public` ) 和私有仓库 ( `Private` ) 两种形式.
          - 最大的公开仓库是 [Docker Hub](https://hub.docker.com).
          - 国内比较流行的仓库
            - 阿里云 (https://xxx.mirror.aliyuncs.com) (可存放私有镜像, 需自行[注册](https://promotion.aliyun.com/ntms/act/kubernetes.html))
            - 网易云 (http://hub-mirror.c.163.com)
          - 私有仓库需自行搭建, 目前比较流行的私有仓库是[Harbor](https://github.com/goharbor/harbor).
    - **镜像**: 一个只读的模板. 可以用来创建Docker容器. 一个镜像可以创建很多容器.
    - **容器**: 容器是镜像创建的运行实例.

4. 通俗理解:  

    <div style="display: flex; width: 100%">
        <div style="width: 15%; margin-left: 20px">
            <!-- 本地图片, 垃圾Gitea无法识别相对路径图片 -->
            <img src="./assets/docker-logo.png" alt="Docker logo">
            <!-- <img src="https://gitea.frp.jinnyu.app/jinnyu/docker/raw/branch/master/assets/docker-logo.png" alt="Docker logo"> -->
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

> ---

##### `Docker` 的安装

1. 安装:

    - CentOS: (yum / rpm) 参考[官方教程](https://docs.docker.com/install/linux/docker-ce/centos/)
    - 通用安装:  
     ```
     curl -fsSL https://get.docker.com -o get-docker.sh
     sudo sh get-docker.sh --mirror Aliyun/AzureChinaCloud
     ```

2. 配置国内镜像:  

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

> ---

##### `Docker` 的优缺点

优点:  

1. 部署方便
  > 在我们最开始学习编程的时候, 搭建环境这一步往往要耗费我们很长时间,   
  > 其中一个小问题也有可能需要很长时间去解决.  
  > 而有了容器之后, 这些都变得非常容易, 我们的开发环境就只是一个或者几个容器镜像的地址,   
  > 最多在再需要一个控制部署流程的执行脚本,   
  > 或者进一步将你的环境镜像以及镜像脚本放入一个 `Git` 项目发布到云端, 需要的时候将它拉取到本地即可. 

2. 部署安全
  > 当我们收到一个bug反馈的时候, 很多时候心里的第一反应都是"在我本地就是好的啊, 是不是你环境有问题？  
  > 这种情况的发生就在于本地环境的不一致(也既我们常说的异构),   
  > 我们在开发环境中调试往往不能保证其它环境的问题, 但却要为此买单.  
  > 有了容器之后, 各个环节采用统一标准(环境), 这种情况将很少发生.  
  > 我们可以通过容器技术将开发环境和测试环境以及生产环境保持版本和依赖上的统一,   
  > 保证我们的代码在一个高度统一的标准上执行,   
  > 而测试环境的同样, 也同样能解决CI(持续集成)流程对环境的要求.  
  > 在分布式技术和扩容需求日益增长的今天, 如果运维能够通过容器技术来进行环境的统一部署,   
  > 不仅在部署的时间上节省不少, 也能把很多人工配置环境过程中产生的失误降到最低. 

3. 隔离性好
  > 不管是开发还是生产, 往往我们一台机器上可能要跑多个服务, 而服务各自依赖的配置又不尽相同.  
  > 假如说两个应用需要使用同一依赖, 或者两个应用需要的依赖之间有一些冲突, 这个时候就很容易出现问题,   
  > 所以同一台服务器上不同应用提供的不同服务, 最好还是将其隔离起来,   
  > 而容器在这一方面就有其天然的优势, 每一个容器就是一个隔离的环境,   
  > 容器内部所提供的服务对环境依赖的要求, 容器可以自内部全部提供,   
  > 这种高内聚的表现可以快速分离有问题的服务, 在一些复杂应用系统中能够实现快速拍错和即时处理. 

4. 快速回滚
  > 容器之前的回滚机制, 一般要基于上个版本的应用重新部署, 且替换掉目前有问题的版本.  
  > 在最初的时代, 可能是一套完成的从开发到部署的流程, 而执行这一套流程往往需要花费很长时间.  
  > 在基于Git的环境中, 可能是回退某个历史提交, 然后重新部署.  
  > 这些跟容器相比, 都不够快, 而且可能会产生新的问题.  
  > 而容器技术天生带有回滚特性, 因为每个历史容器或者镜像都会保存, 而替换某个容器或者镜像是非常快速和简单的.  

5. 成本低
  > 这可能是一个最明显和有用的优点了.  
  > 在容器出现之前, 我们往往会因为构筑一个服务, 就要提供一台服务器或者一台虚拟机.  
  > 服务器的购置成本和运维成本都比较高, 而虚拟机所占用的资源又相对较高,  
  > 相比之下, 容器就小巧轻便的多,  
  > 只需要给一个容器内部构建应用所需的依赖就可以了, 这也是容器技术发展如此迅速的最主要原因. 

6. 管理成本低
  > 随着容器技术的不断普及和发展, 随之而来的容器管理和编排技术也同样得到发展,  
  > 诸如 `Docker Swarm`, `K8S`, `Mesos` 等容器编排工具也在不停的迭代更新,  
  > 这让容器技术在生产环境中拥有了更大的可能性和更多的发挥空间.  
  > 随着大环境的发展, `Docker` 等容器的使用和学习成本也越来越低, 成为更多开发者和企业的选择.

缺点:  

1. 隔离性
  > 基于 `hypervisor` 的虚拟技术, 在隔离性上比容器技术要好, 它们的系统硬件资源完全上虚拟化的.  
  > 当一台虚拟机出现系统级别的问题, 往往不会蔓延到同一宿主机上的其它虚拟机上,  
  > 但是容器就不一样了, 容器之间共享同一个操作系统内核及其它组件,  
  > 所以在受到诸如黑客攻击这种情况的时候, 很容易通过底层操作系统影响的其它容器, 甚至逐个击破, 产生连锁反应,  
  > 当然, 这个问题可以通过部署容器来解决,  
  > 但随之又会产生新的问题, 比如成本增加以及性能问题.

2. 性能
  > 不管是虚拟机还是容器, 都是运用不同的技术对应用本身进行了一定程度的封装与隔离,  
  > 在降低应用和应用之间以及应用和环境之间的耦合性上做了很多努力,  
  > 但是随之而来的, 就会产生更过的网络连接转发和数据交互,  
  > 这在低并发系统上虽然不会很明显, 但当容器需要高并发量支撑的时候, 也就是并发问题成为系统瓶颈的时候,  
  > 容器会将这个问题放大, 所以, 并不是所有的场景都适合容器技术.

3. 存储方案
  > 容器的诞生并不是为 `OS` 抽象服务的.  
  > 这是它和虚拟机最大的区别, 这样的基因意味着容器天生是为应用环境做更多的努力.  
  > 容器的伸缩也是基于容器的这一特性, 而与之相对的, 需要持久化存储方案恰恰相反,  
  > 在数据存储这一点上 `Docker` 容器提供的解决方案是利用 `Volume` 接口(存储卷)形成数据的映射和转移, 以实现数据持久化的目的.  
  > 但是这样同样也会造成一部分资源的浪费和更多的交互, 不管是映射到宿主机上还是网络磁盘, 都是退而求其次的解决方案.  

> 优缺点来自微信公众号 - IT技术精选文摘（ITHK01）  
> 原文出处及转载信息见文内详细说明, 如有侵权, 请联系 yunjia_community@tencent.com 删除.  
> 原始发表时间：2019-06-26 [腾讯云文章链接](https://cloud.tencent.com/developer/article/1457282)

> ---

##### `Docker` 操作命令

- 帮助命令
  - docker version
  - docker info
  - docker --help
      
- 镜像命令
  - docker images \${sub cmd}
    - 解释: 列出本地镜像
      > 同一仓库源可以有多个 `TAG`, 代表这个仓库源的不同版本, 可以使用 `REPOSITORY:TAG` 来定义不用的镜像.
      - REPOSITORY: 镜像仓库源
      - TAG: 镜像标签
      - IMAGE ID: 镜像ID
      - CREATED: 镜像创建时间
      - SIZE: 镜像大小
    - 子命令:
      - -a: 列出本地所有的镜像(含中间映象层)
      - -q: 只显示镜像ID
      - --digests: 显示镜像的摘要(sha256)信息
      - --no-trunc: 显示镜像的完整信息
  - docker search \${sub cmd} \${image name}
    - 解释: 搜索DockerHub上的镜像
      - NAME: 镜像名
      - DESCRIPTION: 镜像描述
      - STARS: 星标数 (类似 `Github` 的星标数)
      - OFFICIAL: 是否为官方镜像
      - AUTOMATED: 是否为自动构建的
    - 子命令:
      - -s: 显示大于指定星数的镜像
      - --no-trunc: 显示镜像的完整信息
      - --automated: 只显示自动构建类型的镜像
  - docker pull \${image name}:\${tag}
    - 解释: 拉取指定镜像
  - docker rmi \${sub cmd} \${image name / image id}:\${tag}
    - 解释: 删除指定镜像
    - 子命令:
      - -f: 强制删除
      
- 容器命令
  - docker run \${sub cmd} \${image name / image id} \${cmd} \${arg}
    - 解释: 新建并启动容器
    - 子命令:
      - --name \${container name} : 为容器指定一个名字
      - -i: 交互模式运行容器, 通常和 `-t` 一起使用.
      - -t: 以伪终端模式运行
      - -d: 后台模式运行容器, 并返回容器ID.
      - -v: 指定数据卷 (-v /宿主机文件路径(绝对路径):/容器文件绝对路径[:文件读写权限])
      - --volumes-from: 容器间共享数据卷 (--volumes-from \${name})
      - -P: 随机分配端口
      - -p: 指定端口
        - ip:host port: container port
        - ip::container port
        - host port: container port
        - container port
  - docker start \${container id}
    - 解释: 启动一个容器
  - docker stop \${container id}
    - 解释: 停止一个容器
  - docker resart
    - 解释: 重启一个容器
  - docker kill \${container id}
    - 解释: 强制停止一个容器
  - docker ps \${sub cmd}
    - 解释: 同 `Linux`的 `ps`
    - 子命令:
      - -a --all : 显示所有容器(默认只显示正在运行的容器)
      - -l --latest : 最后创建的容器
      - -n --last \${num} : 最后创建的\${num}的容器
      - -q : 只显示镜像ID
  - docker rm \${container id}
    - 解释: 删除一个容器
    - 子命令:
      - -f 强制删除一个运行中的容器
  - docker rmi \${image}
    - 解释: 删除一个镜像
    - 子命令:
      - -f 强制删除一个镜像
  - docker logs \${sub cmd} \${container id}
    - 解释: 查看容器日志
    - 子命令:
      - -t: 容器内执行命令的时间戳
      - -f: 跟随最新日志
      - --tail number 显示多少条日志
  - docker top \${container id}
    - 解释: 同 `Linux`的 `top`
  - docker inspect \${container id}
    - 解释: 查看容器内部细节
  - docker attach \${container id}
    - 解释: 将本地标准输入, 输出和错误流附加到正在运行的容器
  - docker exec \${sub cmd} \${container id} \${cmd}
    - 解释: 将\${cmd}传递到容器内运行
    - 子命令:
      - -e: 设置环境变量
      - -d: 后台运行命令
      - -u: 指定用户执行命令 格式: <name|uid>[:<group|gid>]
      - -w: 指定执行目录
      - -i: 交互模式运行容器, 通常和 `-t` 一起使用.
      - -t: 以伪终端模式运行
  - docker cp CONTAINER:SRC_PATH DEST_PATH|- 或 SRC_PATH|- CONTAINER:DEST_PATH
    - 解释: 在容器和宿主机之间复制文件/文件夹  
      > CONTAINER:SRC_PATH DEST_PATH|-  
      > 容器ID:容器文件路径 宿主机目标路径  
      > 使用 “-” 作为源以从stdin中读取tar归档文件, 并将其提取到容器中的目录目的地. 

      > SRC_PATH|- CONTAINER:DEST_PATH  
      > 宿主机文件路径|- 容器ID:容器目标路径  
      > 使用 “-” 作为目标以流式传输tar的tar存档, 容器源到标准输出. 
    - 子命令:
      - -a: 归档模式 复制所有uid/gid信息
  - docker commit -a="author" -m="message" \${container id} \${image name}:\${version}
    - 解释: 以一个镜像为基准进行提交并生成一个容器

> ---

##### `Docker` 镜像

`Docker` 镜像是一种采用 `UnionFS` 为底层的包装产物.
  
  - `UnionFS` (联合文件系统)
    - 是一种分层、轻量级并且高性能的文件系统.  
      它支持**对文件系统的修改作为一次提交来一层叠加**.  
      同时可以将不同目录挂载到用一个虚拟文件系统下.  
      `UnionFS` 是 `Docker` 镜像的基础. 
      镜像可以通过分层来进行**继承**,  基于基础镜像(没有父镜像), 可以制作各种具体的应用镜像. 
   
  - `Docker` 镜像加载原理
    - `bootfs` : 镜像的最底层
    - `rootfs` : 各自发行版的内容

`Docker` 镜像特点
  - `Docker` 镜像都是只读的.  
    当容器启动时, 一个新的可写层被加载到镜像的顶部.
    这一层通常被称为 `容器层` , `容器层` 之下的都叫做 `镜像层`.

> ---

##### `Docker` 容器数据卷

卷是文件或目录, 存在于一个活多个容器中, 由 `Docker` 挂载到容器上.  
卷不属于 `UnionFS` , 所以能绕过 `UnionFS` 提供一些用于从持续存储活共享数据的功能.  
卷的设计目的就是数据的持久化, 完全独立于容器的生命周期, 因此 `Docker` 不会早容器删除时删除其挂载的数据卷.

特点: 
  1. 数据卷可以在容器间共享或者重用.
  2. 卷的更改直接生效.
  3. 数据卷的更改不会包含在镜像的更新中.
  4. 数据卷的声明周期一直持续到没有容器使用它为止.

挂载方式:
  1. 命令行挂载 
     ```
     docker run -v /host/folder:/target/folder ...
     ```
  2. `DockerFile` 挂载
     ```
     ... other config ...
     VOLUME ["/target/folder1", "/target/folder2" ... ]
     ... other config ...
     ```
     宿主机数据卷可以用 `docker inspect ${container id}` 来查询.

> ---

##### `DockerFile`

定义 :  

`DockerFile` 是用来构建 `Docker` 镜像的构建文件, 是有一系列的命令和参数构成的脚本.

基础知识:

1. 每条保留指令都必须大写字母且后边至少要提供一个参数.
2. 指令按照从上到下的方式顺序执行.
3. `#` 表示注释.
4. 每条命令都会创建一个新的镜像层, 并对镜像进行提交.
   
执行流程:

1. `Docker` 从基础镜像运行一个容器 (默认基础镜像 : `scratch` , 类比 `Java` 的 `Object`)
2. 执行一条指令并对容器做出修改
3. 执行类似 `docker commit` 的操作提交一个新的镜像层
4. `Docker` 基于刚提交的镜像运行一个新的容器
5. 继续执行 `DockerFile` 中的下一条指令直到所有指令都执行完成

`DockerFile` 保留指令:

1. `FROM` 基础镜像.
2. `LABEL` 镜像说明
   - 支持多行 `LABEL`
     ``` 
     LABEL com.example.version="0.0.1-beta"
     LABEL vendor1="ACME Incorporated"
     LABEL vendor2=ZENITH\ Incorporated
     LABEL com.example.release-date="2015-02-12"
     LABEL com.example.version.is-production=""
     ```
   - 支持一行 (多个属性之间用空格分隔)
     ```
     LABEL com.example.version="0.0.1-beta" com.example.release-date="2015-02-12"
     ```
   - 支持 `\` 换行
     ```
     LABEL vendor=ACME\ Incorporated \
           com.example.is-beta= \
           com.example.is-production="" \
           com.example.version="0.0.1-beta" \
           com.example.release-date="2015-02-12"
     ``` 
3. `MAINTAINER` 维护者信息.
4. `RUN` 容器构建时需要执行的命令.
5. `EXPOSE` 容器对外暴露的端口.
6. `WORKDIR` 在创建容器后, 终端默认登录进的工作目录. (默认为根目录).
7. `ENV` 用来在构建镜像中设置环境变量.
   - 格式 (支持 `$` 引用)
     ```
     ENV JAVA_HOME /opt/software/jdk  
     ENV PATH $PATH:$JAVA_HOME/bin
     ```
8. `ADD` 将宿主机的文件复制到镜像中, 且 `ADD` 命令会自动处理 `URL` 和解压 `tar` 包.
9. `COPY` 类似 `ADD` , 将宿主机的文件复制到镜像中.
   - `COPY src dest`
   - `COPY [ "src", "dest" ]`
10. `VOLUME` 指定容器数据卷, 用于数据保存和持久化工作.
    - 格式 (宿主机数据卷可以用 `docker inspect ${container id}` 来查询.)
      ```
      VOLUME ["/target/folder1", "/target/folder2" ... ]
      ```
11. `CMD` 指定一个容器启动时要运行的命令
    - `DockerFile` 中可以有多个CMD指令, 但只有最后一个生效, 且 `CMD` 会被 `docker run` 之后的参数替换.
    - `shell` 格式 : `CMD <命令>`
    - `exec` 格式 : `CMD [ "可执行文件", "参数1", "参数2" ... ]`
    - 参数列表格式 : `CMD [ "参数1", "参数2" ... ]` , 在指定了 `ENTRYPOINBT` 指令后, 用 `CMD` 指定具体的参数.
12. `ENTRYPOINT` 指定一个容器启动时要运行的命令
    - `ENTRYPOINT` 和 `CMD` 一样, 都是在指定容器启动程序和参数
13. `ONBUILD` 当构建一个继承于父镜像的镜像时, 父镜像的 `ONBUILD` 在子镜像构建时触发.
