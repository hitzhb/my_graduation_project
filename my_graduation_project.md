# 1. docker

## (1). docker安装

详见https://docs.docker.com/engine/install/ubuntu/

下面的命令可以去掉sudo使用docker，***重启后生效***

```shell
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```

这两条命令据说可以开机后自动启动docker.service,即省去了每次开机运行`sudo service docker start`，经测试，确实可以

```shell
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

进入docker后，要进行一系列操作以便使用ping，ifconfig等命令：

```shell
apt update
apt-get install net-tools
apt-get install iputils-ping
apt-get install vim
```

## (2). docker 常用命令



# 2. ovs

## (1). ovs部署

详见https://docs.openvswitch.org/en/latest/intro/install/general/

部署完成后，每次开机需要启动ovs，使用`sudo su`进入root：

```shell
export PATH=$PATH:/usr/local/share/openvswitch/scripts
ovs-ctl start
ovs-ctl --no-ovs-vswitchd start
ovs-ctl --no-ovsdb-server start
```

***运行后也可以在user命令下使用sudo***

ovs-docker：新版本已经自带ovs-docker了，不用再去github上下载了。

## (2). ovs常用命令



# 3. ryu

## (1). ryu安装

`pip3 install ryu`

安装好后还是会提示`ryu:未找到命令`，根据warning，只需要将路径加入到环境变量即可：

修改.bashrc文件，在最后一行加入

`export PATH=$PATH:/home/zhb/.local/bin`

此时可以运行ryu，但仍然报错：

`ImportError: cannot import name 'ALREADY_HANDLED' from 'eventlet.wsgi' (/home/zhb/.local/lib/python3.9/site-packages/eventlet/wsgi.py)`

经过尝试，这里安装`eventlet==0.30.2`即可：

`pip install eventlet==0.30.2`

***千万不要试图使用GUI，千万不要试图使用GUI，千万不要试图使用GUI！！！***

当然可以使用ryu自带的gui_topology.py，不要使用GUI补丁。

## (2). ryu常用命令



# 4. mininet

## (1). mininet安装



## (2). mininet常用命令



## (3). 利用mininet构建SDN网络

### (i). 在主机内运行docker

采用host模式，安全性不高

采用none模式，运行docker后利用ovs-docker连接ovs

### (ii). 构建0主机

在宿主机上运行docker，采用none模式，利用ovs-docker

# 5. 网络构建流程

1. 启动ovs

2. 启动ryu：`ryu run --verbose --observe-link simple_switch.py gui_topology/gui_topology.py`，simple_switch.py遵循openflow1.0协议，gui_topology.py可以显示拓扑

3. 启动mininet：`sudo mn --controller remote --custom topo-2sw-2host.py --topo mytopo`，`--controller remote`指定远程ryu作为控制器，`--custom topo-2sw-2host.py`可以自己编写该文件实现拓扑的构建

   此时可以看到拓扑：

   ![mytopo_2sw_2host](/home/zhb/Desktop/mytopo_2sw_2host.png)

4. 测试连通性：在mininet中：`h1 ping h2`![h1_ping_h2](/home/zhb/Desktop/h1_ping_h2.png)

   ping 之前ovs的流表：

   ![before_ping_ovs](/home/zhb/Desktop/before_ping_ovs.png)

   ping之后ovs的流表：

   ![after_ping_ovs](/home/zhb/Desktop/after_ping_ovs.png)

   可以发现，ping之后ryu有流表下发，可以看出数据的流向。

5. 在mininet CLI中打开host：`xterm h1`

   在打开的xterm中运行docker：`docker run -itd --net=host --name server -v $(pwd):/server ubuntu:18.04`

   采用host模式发现无法ping通，查看ip地址发现，docker与宿主机的ip地址一致：172.20.65.114，而不是与mininet创建的主机h1：10.0.0.1一致。

test
