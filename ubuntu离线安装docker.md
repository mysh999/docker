## 一、卸载自带的docker （如果有的话）

```bash
$ sudo apt-get purge docker-ce
[sudo] mysh 的密码： 
正在读取软件包列表... 完成
正在分析软件包的依赖关系树       
正在读取状态信息... 完成       
软件包 docker-ce 未安装，所以不会被卸载
下列软件包是自动安装的并且现在不需要了：
  libcurl4 libgsoap-2.8.60 libpython-stdlib libqt5opengl5 libqt5printsupport5 libqt5x11extras5 libsdl1.2debian libvncserver1 python python-minimal python2.7 python2.7-minimal
使用'sudo apt autoremove'来卸载它(它们)。
升级了 0 个软件包，新安装了 0 个软件包，要卸载 0 个软件包，有 164 个软件包未被升级。
```



## 二、下载docker和docker-compose

docker包下载地址：

```htm
https://download.docker.com/linux/ubuntu/dists/
```

![企业微信截图_20201216103609.png](http://ww1.sinaimg.cn/large/007Xg1efgy1glphurtknmj30it0qfabe.jpg)



安装docker

```bash
$ sudo dpkg -i docker-ce-cli_19.03.8~3-0~ubuntu-xenial_amd64.deb 
$ sudo dpkg -i containerd.io_1.2.4-1_amd64.deb 
$ sudo dpkg -i docker-ce_19.03.8_3-0_ubuntu-xenial_amd64.deb 
$ docker -v
Docker version 19.03.8, build afacb8b7f0
```



安装docker-compose

```bash
$ sudo curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

$ sudo chmod +x /usr/local/bin/docker-compose
```

