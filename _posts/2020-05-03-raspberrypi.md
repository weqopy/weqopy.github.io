---
layout: post
title: raspberrypi
date: 2020-05-03 21:18:24 +0800
category: Note
tag: Raspberry
---

* content
{: toc}
---

## 树莓派

### 初始化

#### 系统

参考 https://juejin.im/post/5d4a7fb6e51d4561b416d414

1. 下载系统镜像 https://www.raspberrypi.org/downloads/raspbian/
2. 格式化SD卡
3. 烧录系统

#### 系统文件配置

1. boot目录下创建「ssh」文件，无后缀、空内容

2. boot目录下创建「wpa_supplicant.conf」文件，内容为：

```shell
country=CN
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
ssid="WiFi-A"
psk="12345678"
key_mgmt=WPA-PSK
priority=1 # 隐藏Wi-Fi设置此项
}
```

#### 准备

1. `arp -a` 记录当前网络设备IP地
2. 插入内存卡、连接电源
3. `arp -a` 检查当前网络设备IP地址，查看与之前记录相比多出IP，如 192.168.1.15
4. 或者使用`ping raspberrypi.local`命令获取树莓派IP
5. ssh pi@192.168.1.15 连接，密码默认为raspberry

#### 初始设置

1. 修改密码
2. 设置静态IP，编辑`/etc/dhcpcd.conf`文件

    ```shell
    interface eth0
    static ip_address=192.168.1.22/24
    static routers=192.168.1.1
    static domain_name_servers=192.168.1.1

    interface wlan0
    static ip_address=192.168.1.23/24
    static routers=192.168.1.1
    static domain_name_servers=192.168.1.1
    ```

3. 修改系统更新源

编辑 `/etc/apt/sources.list` 文件，删除原文件所有内容，用以下内容取代：

```shell
deb http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main non-free contrib
deb-src http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main non-free contrib
```

编辑 `/etc/apt/sources.list.d/raspi.list` 文件，删除原文件所有内容，用以下内容取代：

```shell
deb http://mirrors.tuna.tsinghua.edu.cn/raspberrypi/ buster main ui
```

### 内网穿透

参考 [cpolar](https://www.cpolar.com/blog/内网穿透家中的树莓派，开机自启动)

```shell
1. 首先从官网下载最新的cpolar
wget https://www.cpolar.com/static/downloads/cpolar-stable-linux-arm.zip
2. 解压缩
unzip cpolar-stable-linux-arm.zip
3. 将cpolar命令移到 /usr/local/bin目录
sudo mv cpolar /usr/local/bin
4. cpolar进行token认证
登录到cpolar后台，获取到自己的token值,然后复制到命令中，替换xxxxxxxx
cpolar authtoken xxxxxxxxxxxxxxxxxxxxxxx
配置文件会保存在 /home/pi/.cpolar/cpolar.yml，记录下该路径
5. 前台测试
cpolar http 8080
看到Tuunel Status为online说明正常
6.在cpolar配置文件中，添加ssh内网穿透隧道
除了在cpolar命令行中，我们还可以在配置文件中添加多个隧道。这样多个隧道可以同时启动。
vim ~/.cpolar/cpolar.yml
在文件下面，我们编辑成如下内容：
authtoken: xxxxx #认证token
tunnels:
  web:              #隧道名称，用户可以自定义，但多隧道时，不可重复
    addr: 8080      #本地Web站点端口
    proto: http     #协议http
    region: cn_vip  #地区，cn_vip,可选：us,hk,cn,cn_vip
  ssh:              #隧道名称，表示ssh，名称可以自定义
    addr: 22        #端口号为22
    proto: tcp      #协议tcp
    region: cn_vip  #地区，cn_vip，可选:us,hk,cn,cn_vip
7.测试是否配置成功
cpolar start-all
看到Tuunel Status为online说明正常
8. 后台运行cpolar
nohup cpolar start-all -config=/home/pi/.cpolar/cpolar.yml -log=stdout &
9. 验证后台是否运行正常
ps -aux | grep cpolar | grep -v grep
查看进程确认后台运行正常后，杀掉cpolar进程
10. 配置开机自启动脚本
编辑开机脚本
sudo nano /etc/rc.local
在exit 0前面，加入
nohup cpolar start-all -config=/home/pi/.cpolar/cpolar.yml -log=stdout &
11. 重启树莓派
sudo reboot
12. 重启后查看是否成功
ps -aux | grep cpolar | grep -v grep
13. 查看在线的隧道
访问cpolar后台的状态页面:http://dashboard.cpolar.com/status
查看隧道名称
ssh	tcp://URL:PORT
14. 外网环境ssh连接
ssh -p PORT pi@URL
```

### 云盘文件同步

使用`syncthing`

1. 下载安装，本地下载后scp传至服务器。[下载地址](https://github.com/syncthing/syncthing/releases)

2. `tar -zxvf syncthing-***`解压

3. 权限设置`sudo chmod +x syncthing`

4. 初始化执行`./syncthing-***/syncthing`

5. 修改配置文件，允许外网连接

6. ```shell
    sudo vi ~/.config/syncthing/config.xml
    找到：
    <address>127.0.0.1:8384</address>
    将其改为：
    <address>0.0.0.0:8384</address>
    ```

7. 后台运行、自启动设置`sudo vim /etc/init.d/syncthing`

8. ```shell
    #!/bin/sh

    ### BEGIN INIT INFO
    # Provides:          Syncthing
    # Required-Start:    $local_fs $remote_fs $network
    # Required-Stop:     $local_fs $remote_fs $network
    # Default-Start:     2 3 4 5
    # Default-Stop:      0 1 6
    # Short-Description: Syncthing
    # Description:       Syncthing is for backups
    ### END INIT INFO

    # Documentation available at
    # http://refspecs.linuxfoundation.org/LSB_3.1.0/LSB-Core-generic/LSB-Core-generic/iniscrptfunc.html
    # Debian provides some extra functions though
    . /lib/lsb/init-functions

    DAEMON_NAME="syncthing"
    # 使用pi用户
    DAEMON_USER=pi
    # 注意！这里的DAEMON_PATH是syncting所在的路径，每个人的安装方式、版本不一样，这个路径都会有所区别，请自行修改
    DAEMON_PATH="/home/pi/syncthing-linux-arm-v1.4.2/syncthing"
    DAEMON_OPTS=""
    DAEMON_PWD="${PWD}"
    DAEMON_DESC=$(get_lsb_header_val $0 "Short-Description")
    DAEMON_PID="/var/run/${DAEMON_NAME}.pid"
    DAEMON_NICE=0
    DAEMON_LOG='/var/log/syncthing'

    [ -r "/etc/default/${DAEMON_NAME}" ] && . "/etc/default/${DAEMON_NAME}"

    do_start() {
      local result

        pidofproc -p "${DAEMON_PID}" "${DAEMON_PATH}" > /dev/null
        if [ $? -eq 0 ]; then
            log_warning_msg "${DAEMON_NAME} is already started"
            result=0
        else
            log_daemon_msg "Starting ${DAEMON_DESC}" "${DAEMON_NAME}"
            touch "${DAEMON_LOG}"
            chown $DAEMON_USER "${DAEMON_LOG}"
            chmod u+rw "${DAEMON_LOG}"
            if [ -z "${DAEMON_USER}" ]; then
                start-stop-daemon --start --quiet --oknodo --background \
                    --nicelevel $DAEMON_NICE \
                    --chdir "${DAEMON_PWD}" \
                    --pidfile "${DAEMON_PID}" --make-pidfile \
                    --exec "${DAEMON_PATH}" -- $DAEMON_OPTS
                result=$?
            else
                start-stop-daemon --start --quiet --oknodo --background \
                    --nicelevel $DAEMON_NICE \
                    --chdir "${DAEMON_PWD}" \
                    --pidfile "${DAEMON_PID}" --make-pidfile \
                    --chuid "${DAEMON_USER}" \
                    --exec "${DAEMON_PATH}" -- $DAEMON_OPTS
                result=$?
            fi
            log_end_msg $result
        fi
        return $result
    }

    do_stop() {
        local result

        pidofproc -p "${DAEMON_PID}" "${DAEMON_PATH}" > /dev/null
        if [ $? -ne 0 ]; then
            log_warning_msg "${DAEMON_NAME} is not started"
            result=0
        else
            log_daemon_msg "Stopping ${DAEMON_DESC}" "${DAEMON_NAME}"
            killproc -p "${DAEMON_PID}" "${DAEMON_PATH}"
            result=$?
            log_end_msg $result
            rm "${DAEMON_PID}"
        fi
        return $result
    }

    do_restart() {
        local result
        do_stop
        result=$?
        if [ $result = 0 ]; then
            do_start
            result=$?
        fi
        return $result
    }

    do_status() {
        local result
        status_of_proc -p "${DAEMON_PID}" "${DAEMON_PATH}" "${DAEMON_NAME}"
        result=$?
        return $result
    }

    do_usage() {
        echo $"Usage: $0 {start | stop | restart | status}"
        exit 1
    }

    case "$1" in
    start)   do_start;   exit $? ;;
    stop)    do_stop;    exit $? ;;
    restart) do_restart; exit $? ;;
    status)  do_status;  exit $? ;;
    *)       do_usage;   exit  1 ;;
    esac
    ```

9. 设置权限及打开自启动

10. ```shell
    #添加执行权限
    sudo chmod +x /etc/init.d/syncthing
    #添加自启启动
    sudo update-rc.d syncthing defaults
    ```

11. GUI 及客户端设置

    [GUI页面](http://192.168.1.23:8384)

    [macOS客户端](https://github.com/syncthing/syncthing-macos)
