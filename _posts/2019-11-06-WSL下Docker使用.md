# WSL下Docker使用

## WSL 简介
- WSL 是 Window Subsystem for Linux 简称, 是Windows 和 Canonical 在windows 10 下面推出的一个Ubuntu 发行版
- 它可以在windows10 下运行原生的Linux 程序

### 相比于虚拟机的和物理机的优势

 - 相比于虚拟机, 占用的 内存 和 CPU 资源更少, 可以直接访问windows 上文件资源, 不需要建立Samba服务或共享盘符
 - 相比于物理机, 重新安装 UWP(universal windows platform) 应用即可, 可以随时快速重新安装系统, 所以不建议在此运行数据库软件和保存关键性数据
 - 相比于MinGW Cygwin, 有更好的兼容性和原生体验

### 安装WSL
 - 打开 powershell ,输入神秘代码 `Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux`, 重启电脑
 - 打开window10 应用商店,搜索Linux, 点击安装即可
 - Win10 1703 版本之前, 需要打开开发者选项
 - 命令行设置用户名和密码即可使用
 - windows 下输入bash 也可以进入WSL 

## 使用前的一些设置

### 更换仓库源

#### 备份官方仓库

```shell
sudo mv sources.list sources.list.bak
```

#### 设置新仓库源
```shell
sudo vim source.list
```

```shell
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
```
- 发行代号是否匹配(lsb_release -a 或 screenfetch)

#### [安装docker](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

详细见官方文档

1.权限问题

```shell
sudo service docker start

cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running
```
- WSL 需要以 `管理员身份` 运行, 否者 WSL 下sudo 无法拿到最高权限

2. 版本兼容问题
windows 1803 以下: `17.03.0` ~ `17.09.0`
windows 1809 可用: `17.03.0` ~ `18.06.1`

```shell
# docker: Error response from daemon: OCI runtime create failed: container_linux.go:346: starting container process caused "process_linux.go:319: getting the final child's pid from pipe caused \"EOF\"": unknown.

# sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu --allow-downgrades
```

3. cgroup

```shell
Error starting daemon: Devices cgroup isn't mounted

sudo apt -y install cgroupfs-mount
sudo cgroupfs-mount

```

4. 创建网桥问题

```shell
Error starting daemon: Error initializing network controller: error obtaining controller instance: failed to create NAT chain: iptables failed: iptables -t nat -N DOCKER: iptables v1.6.0: can't initialize iptables table `nat': Table does not exist (do you need to insmod?)
Perhaps iptables or your kernel needs to be upgraded.

Error starting daemon: Error initializing network controller: Error creating default "bridge" network: Failed to Setup IP tables: Unable to enable NAT rule:  (iptables failed: iptables --wait -t nat -I POSTROUTING -s 172.19.0.0/16 ! -o docker0 -j MASQUERADE: iptables: Invalid argument. Run `dmesg' for more information.

dockerd --iptables=false

```
- 注意windows 下防火墙问题

5.进入容器

```shell
docker exec -it c729e7957506 /bin/sh
OCI runtime exec failed: exec failed: container_linux.go:348: starting container process caused "could not create session key: function not implemented": unknown

sudo nsenter -p -i -u -m -n -t `docker inspect -f {{.State.Pid}} nginx` sh

# vim .bashrc
function docker-exec {
    name=$1
    shift
    sudo nsenter -p -i -u -m -n -t `docker inspect -f {{.State.Pid}} ${name}` sh

}


```



#### 验证docker 环境

```shell
# docker run -dit --name nginx -p 3000:80 nginx:alpine
```




















