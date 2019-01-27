# shadowsocks + Vultr 实现科学上网

> 写这篇文章的原因是，由于我个人的Google cloud被Google莫名其妙的关停，导致我科学上网受阻，所以写一下这篇教程。

## 购买 vultr

首先得需要一个VPS，这里我用的是[Vultr](https://www.vultr.com/?ref=7804523)，经过对比Vultr的性价比很高：速度、价格（最便宜的2.5刀）、流量都很不错，如果只用来部署Shadowsocks搓搓有余，还能额外建个小网站。现在以用支付宝和微信来支付。

### 注册vultr账号

[点击注册](https://www.vultr.com/?ref=7804523)

[或点击这里](https://www.vultr.com/?ref=7804552-4F)

![](https://img2018.cnblogs.com/blog/1106982/201901/1106982-20190126223012609-992508186.png)



### 充值

Vultr可以选支付宝来充值，支付宝最低充值10刀

![](https://img2018.cnblogs.com/blog/1106982/201901/1106982-20190126223055512-597161249.png)

### 添加服务器

充值完成后，点Servers，点右上角的加号来添加服务器

![](https://img2018.cnblogs.com/blog/1106982/201901/1106982-20190126223216488-906635838.png)

然后选择服务器位置、配置以及系统版本，至于哪个地方比较合适，[这边有个网址大家可以测试下下载速度]()

![](https://img2018.cnblogs.com/blog/1106982/201901/1106982-20190126223353706-2083523159.png)

系统的话，我选择64位的Ubuntu18.04（系统版本要一样，不一样的话有可能搭建失败）

![](https://img2018.cnblogs.com/blog/1106982/201901/1106982-20190126223431874-126843310.png)

接下来是选择价格，我这边2.5刀的，选完直接点右下角的购买就可以了，其他选项可以不管。Vultr购买完不会马上扣你钱，它是按小时收费的，用多久收多少，不用可以直接停掉

![](https://img2018.cnblogs.com/blog/1106982/201901/1106982-20190126224050025-263194975.png)

选好配置，然后点击购买

![](https://img2018.cnblogs.com/blog/1106982/201901/1106982-20190126224222489-1347409855.png)

![](https://img2018.cnblogs.com/blog/1106982/201901/1106982-20190126225057870-622576355.png)

服务器购买完成后，等几分钟，等创建好之后点进去看详情，记住IP地址、用户名和密码。

![](https://img2018.cnblogs.com/blog/1106982/201901/1106982-20190126225250792-1007761515.png)

![](https://img2018.cnblogs.com/blog/1106982/201901/1106982-20190126225959660-1117246403.png)

* 提醒大家不要买2.5$的那个，它只支持ipv6协议，很恶心！

* 不要选用日本的机房


## 部署

1. 首先我们要做的是连接服务器，我们需要一个ssh客户端来连接，我这边用的是xshell（大家百度xshell自行下载），打开xshell后输入服务器的ip地址点open就可以了，然后输入用户名和密码（鼠标右键是粘贴）就进去到你的远程服务器了。

2. 安装pip和几个依赖包，安装过程遇到Y/n的一律输入Y（按顺序执行下面命令）：

> apt-get install python-pip python-gevent python-m2crypto

> pip install --upgrade setuptools

3. 安装Shadowsocks：

> pip install shadowsocks

出现Successfully installed shadowsocks-XXX说明安装成功了

![](https://img2018.cnblogs.com/blog/1106982/201901/1106982-20190126235551514-1470805275.png)


4. 按顺序执行下面命令，在/etc目录下新建文件夹“shadowsocks”,然后在shadowsocks文件夹下新建文件“config.json”：

> mkdir /etc/shadowsocks
> touch /etc/shadowsocks/config.json
> vim /etc/shadowsocks/config.json

输入完上面命令之后出现的页面，就相当于Windows中的记事本。在这个视图中有如下几个按键需要记住“i”：按键盘上的i键，窗口最底下显示“insert”，表示当前文件可编辑。“Esc”：编辑完之后按Esc退出编辑模式。“:”：半角的冒号，在非编辑模式下按键盘上的冒号（半角），可以进入输入命令的模式。“w”：在命令模式中输入w并回车，窗口最下显示“written”，表示所做的更改已保存。“q”：在命令模式中输入q并回车，可以退出当前的编辑器。
config.json的内容如下：

```
{
    "server":"0.0.0.0",
    "server_port":8888,
    "password":"your_password",
    "timeout":600,
    "method":"aes-256-cfb",
    "fast_open": false
}

```

"server"：是你Vultr服务器的ip地址

"server_port"和"password"可以根据自己的要求设定

如果需要同时开多个端口，config.json的内容可以设置如下：

```
{
    "server":"0.0.0.0",
    "port_password": {
        "8888": "your_password1",
        "8889": "your_password2"
    },
    "timeout":600,
    "method":"aes-256-cfb",
    "fast_open": false
}

```

5. 修改文件openssl.py,我这边python的版本是2.7，其他版本的请修改至对应路径：

vim /usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/openssl.py

替换文中libcrypto.EVP_CIPHER_CTX_cleanup 为libcrypto.EVP_CIPHER_CTX_reset 共两处（大概52行和111行位置），并保存

![](https://img2018.cnblogs.com/blog/1106982/201901/1106982-20190127000943272-1815218428.png)


6. 执行以下命令启动Shadowsocks：

> ssserver -c /etc/shadowsocks/config.json -d start

停止Shadowsocks执行如下命令（此步不需要执行）：

> ssserver -c /etc/shadowsocks/config.json -d stop

7. 设置Shadowsocks开机自启动

执行下面的命令，创建shadowsocks.servic文件：

> touch /etc/systemd/system/shadowsocks.service

> vim /etc/systemd/system/shadowsocks.service

shadowsocks.service的内容如下：

```
[Unit]
Description=Shadowsocks
After=network.target

[Service]
Type=forking
PIDFile=/run/shadowsocks/server.pid
PermissionsStartOnly=true
ExecStartPre=/bin/mkdir -p /run/shadowsocks
ExecStartPre=/bin/chown root:root /run/shadowsocks
ExecStart=/usr/local/bin/ssserver --pid-file /var/run/shadowsocks/server.pid -c /etc/shadowsocks/config.json -d start
Restart=on-abort
User=root
Group=root
UMask=0027

[Install]
WantedBy=multi-user.target

```

设置文件权限：

> chmod 755 /etc/systemd/system/shadowsocks.service

启动服务：

> systemctl start shadowsocks

> systemctl enable shadowsocks

## Final

到此，你的×××已经搭好了，你现在只要去下载shadowsocks的客户端填上Config.json中的ip地址、端口号、以及密码就可以了。

大家如果有遇到问题可以关注本博客其他文章，以后会持续更新

`shadowsocks客户端下载地址：`

```
Windows   
https://github.com/shadowsocks/shadowsocks-windows/releases   

Mac OS X   
https://github.com/shadowsocks/ShadowsocksX-NG/releases  

linux   
https://github.com/shadowsocks/shadowsocks-qt5/wiki/Installation   
https://github.com/shadowsocks/shadowsocks-qt5/releases 

iOS   
https://itunes.apple.com/app/apple-store/id1070901416?pt=2305194&ct=shadowsocks.org&mt=8   
https://github.com/shadowsocks/shadowsocks-iOS/releases  

https://play.google.com/store/apps/details?id=com.github.shadowsocks   
https://github.com/shadowsocks/shadowsocks-android/releases


```
