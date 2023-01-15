# 基于 VirtualBox 的网络攻防基础环境搭建

## 实验目的

- 掌握 VirtualBox 虚拟机的安装与使用；

- 掌握 VirtualBox 的虚拟网络类型和按需配置；

- 掌握 VirtualBox 的虚拟硬盘多重加载；

## 实验环境

- VirtualBox 虚拟机

- 攻击者主机（Attacker）：Kali Rolling 2019.2

- 网关（Gateway, GW）：Debian Buster

- 靶机（Victim）：From Sqli to shell / xp-sp3 / Kali

## 实验要求

- 虚拟硬盘配置成多重加载；

- 搭建满足作业拓扑图所示的虚拟机网络拓扑；

- 完成以下网络连通性测试：
    - 靶机可以直接访问攻击者主机
    - 攻击者主机无法直接访问靶机
    - 网关可以直接访问攻击者主机和靶机
    - 靶机的所有对外上下行流量必须经过网关
    - 所有节点均可以访问互联网


## 实验过程

### 配置虚拟硬盘多重加载

（以`kali`为例）

- 打开`管理>虚拟介质管理`，选择`kali`对于的`vdi`文件，右键选择释放

![shifang](img/释放.png)

- 将类型从普通改为多重加载，然后应用

![duochongjiaz](img/多重加载.png)

- 添加虚拟硬盘

![xuniyingpan](img/虚拟硬盘.png)

- `Debian`和`XP`对应的虚拟硬盘进行同样操作，查看结果

![dajian](img/搭建完成.png)

### 根据拓扑结构配置网络

- 配置`gateway`的网卡，四张网卡的状态分别为：

  - NAT网络，使网关可访问攻击者主机

  - 仅主机（Host-Only）网络，进行网卡设置

  - 内部网络intnet1，搭建局域网1

  - 内部网络intnet2，搭建局域网2

![gateway](img/gateway.png)

- 配置攻击者`attacker`的网络状态为:
  - NAT

  - 两块不同的Host-Only网卡

![attacker](img/attacker.png)

- 4台victim靶机分别在两个局域网内，仅需配置内部网络的一张网卡即可：

  - victim-kali-1和victim-xp-1在局域网intnet1内

  - victim-debian-2和victim-xp-2在局域网intnet2内

![victim](img/victim.png)

### 连通性测试

*获取IP地址时需要打开`Debian-gateway`，也即打开网关。*

查看IP指令
```
Windows XP : ipconfig
Debian: ip addr show 
kali:  ip add
```

|主机|IP地址|
| ---- | ---- |
|kali-attacker|10.0.2.15|
|kali-victim-1|172.16.111.117|
|xp-victim-1|172.16.111.113|
|debian-victim-2|172.16.222.136|
|xp-victim-2|172.16.222.131|

#### 靶机可以直接访问攻击者主机
- `Internet1`内的靶机

![xp-1](img/v-1-a.png)

- `Internet2`内的靶机

![debian-2](img/v-2-a.png)


#### 攻击者主机无法直接访问靶机

对局域网1内的`xp-victim-1`和局域网2内的`debian-victim-2`

![cannot](img/a-cannot-v.png)


#### 网关可以直接访问攻击者主机和靶机

- 网关可以访问攻击者主机（`kali-attacker`）：

![gateway-attacker](img/gateway-attacker.png)

- 网关可以访问局域网1内的靶机（`xp-victim-1`）：
  
![gateway-intnet-1](img/gateway-intnet-1.png)

- 网关可以访问局域网2内的靶机（`debian-victim-2`）：

![gateway-intnet-2](img/gateway-intnet-2.png)

#### 靶机的所有对外上下行流量必须经过网关

靶机的所有对外上下行流量必须经过网关,在网关对应的主机上安装tcpdump和tmux

```
apt install tcpdump
apt update && apt install tmux
```
网关抓包指令
```
sudo tcpdump -c 5
```
- 抓包
    - 局域网1内的靶机
    ![tcp-1](img/tcpintnet1.png)

    - 局域网2内的靶机
    ![tcp-2](img/tcpintnet2.png)

- `wireshark`进行分析

利用vscode远程连接网关对应的主机，使用指令`tcpdump -i enp0s9 -n -w 202301.pcap`将抓取到的包放到pcap文件中，在本地用wireshark进行分析

![wireshark](img/wireshark.png)

对应的ip数据符合靶机和目标网址等信息，说明靶机的所有上下行流量必须经过网关

#### 所有节点均可以访问互联网
  
- 网关可以正常访问互联网：
  
![gateway can surf internet](img/gateway-on.png)

- `attacker`可以正常访问互联网：

![attacker can surf internet](img/attacker-on.png)

- `Internet1` 中的靶机可以直接访问互联网：

![victim1 can surf internet](img/intnet1-on.png)

- `Internet2` 中的靶机可以直接访问互联网：

![victim2 can surf internet](img/intnet2-on.png)


## 实验问题

网关在ping`xp-victim-1`时无法ping通，是Windows防火墙的原因，关闭即可

![fanghuoqiang](img/防火墙.png)

## 参考链接

[Debian服务器root用户远程SSH登录](https://blog.csdn.net/u013541325/article/details/121970885)
