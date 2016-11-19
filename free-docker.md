# 使用樱花免费Docker搭建V2Ray + KCPTun实现科学上网

Last Update: 2016-11-19

**本文作者不为本文任何用途承担责任**

## 1 准备

在完成下列步骤之前，我们假设您：

* 有一个可以连接网络的计算机
* 有一个电子邮箱帐号

** 提示：本教程是针对Windows的教程**

## 2 注册免费Docker帐户

访问 https://app.arukas.io/ 并使用您的邮箱注册一个帐户。

## 3 登陆帐户 & 部署Centos7

在 https://app.arukas.io/ 登陆注册的帐户并且单击“Create a new application”

App Name随意填写，**Image填写tutum/centos:centos7**，Memory选择512MB，Port加3个，**分别填写22(tcp),999(tcp),998(udp)**。其他项默认。

选择“Create Application”，并且等待2分钟。

刷新页面，发现应用图标变绿后，点击应用名称进入应用管理页面。选择Watch，会显示root密码，**请记录root密码！**

然后记录Port栏所有内容，例如(p1,p2,p3为了安全遂隐藏)

```

http://seaof-153-125-231-176.jp-tokyo-02.arukascloud.io:p1 (22/tcp)
http://seaof-153-125-231-176.jp-tokyo-02.arukascloud.io:p2 (999/tcp)
http://seaof-153-125-231-176.jp-tokyo-02.arukascloud.io:p3 (998/udp)

```

安装一个ssh工具（例如putty, XShell）并且连接到Docker容器。

> 提示：IP地址就是你刚才记录的(去掉http:// )，例如`seaof-153-125-231-176.jp-tokyo-02.arukascloud.io` , 端口就是p1。

## 4 安装基本工具 & 部署V2Ray和KCPTun

连接到Docker容器后，输入`yum install wget vim`安装wget和vim

完成后，输入`bash <(curl -L -s https://raw.githubusercontent.com/v2ray/v2ray-core/master/release/install-release.sh)` 等待完成。

完成后，输入`wget https://github.com/xtaci/kcptun/releases/download/v20161118/kcptun-linux-amd64-20161118.tar.gz` 等待完成

完成后，输入`tar -zxvf kcptun-linux-amd64-20161118.tar.gz` 等待完成

完成后，输入`vim v2kcp.json` 并按`i`，复制以下内容并粘贴。

```

{
"listen": ":998",
"target": "127.0.0.1:999",
"key": "x",
"crypt": "aes-128",
"mode": "fast2",
"mtu": 1400,
"sndwnd": 1024,
"rcvwnd": 128
}

```

然后按`ESC`键（键盘右上角），然后输入`:wq`然后回车。

完成后，输入`vim config.json` 并按`i`，复制以下内容并粘贴。

```

{
  "log" : {
    "access": "/root/v2ray-access.log",
    "error": "/root/v2ray-error.log",
    "loglevel": "debug"
  },
  "inbound": {
    "port": 999,
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "3b129dec-72a3-4d28-aeee-028a0fe86e22",
          "level": 1
        }
      ]
    }
  },
  "outbound": {
    "protocol": "freedom",
    "settings": {}
  },
  "outboundDetour": [
    {
      "protocol": "blackhole",
      "settings": {},
      "tag": "blocked"
    }
  ],
  "routing": {
    "strategy": "rules",
    "settings": {
      "rules": [
        {
          "type": "field",
          "ip": [
            "0.0.0.0/8",
            "10.0.0.0/8",
            "100.64.0.0/10",
            "127.0.0.0/8",
            "169.254.0.0/16",
            "172.16.0.0/12",
            "192.0.0.0/24",
            "192.0.2.0/24",
            "192.168.0.0/16",
            "198.18.0.0/15",
            "198.51.100.0/24",
            "203.0.113.0/24",
            "::1/128",
            "fc00::/7",
            "fe80::/10"
          ],
          "outboundTag": "blocked"
        }
      ]
    }
  }
}

```

然后按`ESC`键（键盘右上角），然后输入`:wq`然后回车。

完成后，输入`nohup /usr/bin/v2ray -config /root/config.json &`然后按两次回车。

完成后，输入`nohup ./server_linux_amd64 -c v2kcp.json &`然后按两次回车。

至此服务端配置完成，键入`exit`并关闭ssh工具。

## 5 客户端安装V2Ray和KCPTun

在任意位置新建目录，把以下链接下载到目录内



```
https://github.com/xtaci/kcptun/releases/download/v20161118/kcptun-windows-amd64-20161118.tar.gz (如果你是
