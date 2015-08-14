---
layout: post
category : lessons
title: "How To Set Up an OpenVPN Server on Ubuntu 14.04"
tagline: "Supporting tagline"
tags : [openvpn]
---

## Step 1 --安装与配置OpenVPN 服务器环境

### openVPN 配置
在安装之前，先更新Ubuntu仓库列表

```
apt-get update
```

然后安装OpenVPN 和Easy-RSA

```
apt-get install openvpn easy-rsa
```

解压VPN服务器配置示例文件到`/etc/openvpn`

```
gunzip -c /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz > /etc/openvpn/server.conf
```

解压之后，编辑`server.conf`

```
vim /etc/openvpn/server.conf
```

找到下面内容

```
# Diffie hellman parameters.
# Generate your own with:
#   openssl dhparam -out dh1024.pem 1024
# Substitute 2048 for 1024 if you are using
# 2048 bit keys.
dh dh1024.pem
```

将`dh1024.pem`修改为：

```
dh2048.pem
```

这样，在生成服务端和客户端key的时候会使用加倍的RSA key。

还是在`server.conf`文件，找到下面的内容：

```
# If enabled, this directive will configure
# all clients to redirect their default
# network gateway through the VPN, causing
# all IP traffic such as web browsing and
# and DNS lookups to go through the VPN
# (The OpenVPN server machine may need to NAT
# or bridge the TUN/TAP interface to the internet
# in order for this to work properly).
;push "redirect-gateway def1 bypass-dhcp"
```

取消注释 `push "redirect-gateway def1 bypass-dhcp"`,这样VPN服务器会把客户端流量传递到它的目的地。改完之后，如下：

```
push "redirect-gateway def1 bypass-dhcp"
```

找到下面的内容：

```
# Certain Windows-specific network settings
# can be pushed to clients, such as DNS
# or WINS server addresses.  CAVEAT:
# http://openvpn.net/faq.html#dhcpcaveats
# The addresses below refer to the public
# DNS servers provided by opendns.com.
;push "dhcp-option DNS 208.67.222.222"
;push "dhcp-option DNS 208.67.220.220"
```

取消注释 `push "dhcp-option DNS 208.67.222.222" `和` push "dhcp-option DNS 208.67.220.220"`。改完之后，如下：

```
push "dhcp-option DNS 208.67.222.222"
push "dhcp-option DNS 208.67.220.220"
```

当客户端进行DNS查询时，上面的改动指示服务器推送OpenDNS给客户端。这样可以避免DNS请求泄露到VPN链接之外。 However, it's important to specify desired DNS resolvers in client devices as well. 尽管OpenVPN默认使用OpenDNS，你可以使用其它喜欢的DNS服务。

`server.conf`最后需要修改的内容如下：

```
# You can uncomment this out on
# non-Windows systems.
;user nobody
;group nogroup
```

取消 `user nobody` 和 `group nogroup`。改完之后，如下：

```
user nobody
group nogroup
```

默认情况下，OpenVPN 以root身份运行。我们将其配置为`nobody`用户，`nogroup`组。

保存退出。

### Packet Forwarding

这是sysctl 设置，告诉服务器内核把来自客户端的流量转发到互联网。否则，流量会停在服务器。在运行时启用packet forwarding 使用下面的命令：

```
echo 1 > /proc/sys/net/ipv4/ip_forward
```

还需要把改动持久化，这样服务器重启后，还可以forward

```
vim /etc/sysctl.conf
```

在文件内容顶部，会看到：

```
# Uncomment the next line to enable packet forwarding for IPv4
#net.ipv4.ip_forward=1
```

取消注释 `net.ipv4.ip_forward` 。改完之后，如下：

```
# Uncomment the next line to enable packet forwarding for IPv4
net.ipv4.ip_forward=1

```

保存并退出。

### Uncomplicated Firewall (ufw)

ufw是iptables的前端，设置ufw很简单。Ubuntu 14.04 默认包含ufw。所以我们只需编辑几条规则和配置，然后开启firewall。更详细使用，参见：[ How To Setup a Firewall with UFW on an Ubuntu and Debian Cloud Server.](https://www.digitalocean.com/community/articles/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server)


首先，设置ufw允许ssh。执行下面的命令：

```
ufw allow ssh
```

OpenVPN使用UDP，还要允许1194端口的UDP流量。

```
ufw allow 1194/udp
```

ufw的forward 策略也需要设置。编辑ufw的主配置文件：

```
vim /etc/default/ufw
```

找到`DEFAULT_FORWARD_POLICY="DROP"`。将DROP改为ACCEPT。如下：

```
DEFAULT_FORWARD_POLICY="ACCEPT"
```

Next we will add additional ufw rules for network address translation and IP masquerading of connected clients.

```
vim /etc/ufw/before.rules
```

修改内容如下：添加OPENVPN RULES 块。

```
#
# rules.before
#
# Rules that should be run before the ufw command line added rules. Custom
# rules should be added to one of these chains:
#   ufw-before-input
#   ufw-before-output
#   ufw-before-forward
#

# START OPENVPN RULES
# NAT table rules
*nat
:POSTROUTING ACCEPT [0:0] 
# Allow traffic from OpenVPN client to eth0
-A POSTROUTING -s 10.8.0.0/8 -o eth0 -j MASQUERADE
COMMIT
# END OPENVPN RULES

# Don't delete these required lines, otherwise there will be errors
*filter

```

改完配置，就可以启用ufw了。

```
ufw enable
```

运行输出：

```
Command may disrupt existing ssh connections. Proceed with operation (y|n)?
```

输入 `y`。输出如下：

```
Firewall is active and enabled on system startup
```

查看ufw 主防火墙规则：

```
ufw status
```

输出结果:

```
Status: active

To                         Action      From
--                         ------      ----
22                         ALLOW       Anywhere
1194/udp                   ALLOW       Anywhere
22 (v6)                    ALLOW       Anywhere (v6)
1194/udp (v6)              ALLOW       Anywhere (v6)
```

## Step 2 — Creating a Certificate Authority and Server-Side Certificate & Key


OpenVPN 利用证书加密流量

### Configure and Build the Certificate Authority

接下来该建立我们自己的Certificate Authority (CA) 并且给OpenVPN服务器生成证书和key。OpenVPN支持基于证书的双向验证：换句话说双方建立信任之前，客户端必须验证服务端的证书，服务端也必须验证客户端的证书。

首先拷贝Easy-RSA 生成脚本

```
cp -r /usr/share/easy-rsa/	 /etc/openvpn
```

创建key的保存目录

```
mkdir /etc/openvpn/easy-rsa/keys
```

Easy-RSA 有个变量文件，我们可以编辑它，用它来创建属于我们的证书。This information is copied to the certificates and keys, and will help identify the keys later.

```
vim /etc/openvpn/easy-rsa/vars
```

修改下面变量的值：

```
export KEY_COUNTRY="US"
export KEY_PROVINCE="TX"
export KEY_CITY="Dallas"
export KEY_ORG="My Company Name"
export KEY_EMAIL="sammy@example.com"
export KEY_OU="MYOrganizationalUnit"
```

还是在`vars`文件，修改下面的这行。简单起见，我们使用`server`作为key name。如果你用了其它的名字，你需要修改OpenVPN的配置文件that reference server.key and server.crt。

```
export KEY_NAME="server"
```

我们需要生成Diffie-Hellman parameters：

```
openssl dhparam -out /etc/openvpn/dh2048.pem 2048
```

切换目录

```
cd /etc/openvpn/easy-rsa
```

初始化PKI(Public Key Infrastructure)。注意命令前面的点和空格。

```
. ./vars
```

输出如下：因为我们还没有在key目录中生成任何东西，所以不用在意

```
NOTE: If you run ./clean-all, I will be doing a rm -rf on /etc/openvpn/easy-rsa/keys
```

现在，清理工作目录

```
./clean-all
```

最后的命令用来创建certificate authority (CA)。输出结果会让你确认在Easy-RSA 变量文件中的变量值

```
./build-ca
```

### Generate a Certificate and Key for the Server


还是在`/etc/openvpn/easy-ras`目录中，执行下面的命令生成服务器的key。注意 `server`要和你在Easy-RSA `vars`文件中设置的`export KEY_NAME`的值一致。

```
./build-key-server server
```

输出结果和 执行`./build-ca`时类似。只不过多了两个：

```
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

都可以为空，直接回车就行。

还有两个需要确认的提示：

```
Sign the certificate? [y/n]
1 out of 1 certificate requests certified, commit? [y/n]
```

最后的输出如下：

```
Write out database with 1 new entries
Data Base Updated
```

### Move the Server Certificates and Keys

OpenVPN 需要在`/etc/openvpn` 目录中找到服务端CA，证书和key。我们需要移动它们到正确的位置。

```
cp /etc/openvpn/easy-rsa/keys/{server.crt,server.key,ca.crt} /etc/openvpn
```

确认下是否拷贝成功：

```
ls /etc/openvpn
```

应该能看到服务端的证书和key

此时，OpenVPN 服务器可以跑起来了。运行并检查状态：

```
service openvpn start
service openvpn status
```

输出结果：

```
VPN 'server' is running
```

祝贺你，你的OpenVPN服务已经成功运行。如果状态显示VPN没有运行，那么查看`/var/log/syslog`文件，查看错误。如果错误如下：

```
Options error: --key fails with 'server.key': No such file or directory
```

上面的错误指示： `server.key` 没有拷贝到`/etc/openvpn `。重新拷贝。


## Step 3 — Generate Certificates and Keys for Clients

### Key and Certificate Building








和生成服务端key 类似。切换到目录 `/etc/openvpn/easy-rsa`:

```
./build-key client1
```

再次，你会被询问变量值。和下面的提示：

```
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

仍是留空，直接回车。

还有下面两个需要确认：

```
Sign the certificate? [y/n]
1 out of 1 certificate requests certified, commit? [y/n]
```

如果成功生成了key ,输出结果如下：

```
Write out database with 1 new entries
Data Base Updated
```

客户端配置文件的样例也应该拷贝到Easy-RSA key 目录。稍后会下载到客户端，用作模板进行编辑。在拷贝过程中，我们修改了后缀，将`client.conf`改为了`client.ovpn`。因为客户端需要使用`.ovpn`。

```
cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf /etc/openvpn/easy-rsa/keys/client.ovpn
```

重复上面的步骤，给更多的客户端生成key。

### Transferring Certificates and Keys to Client Devices

我们创建的客户端证书和key,保存在服务端的 `/etc/openvpn/easy-rsa/keys` 目录

对于每个客户端，我们需要将客户端证书、key和profile模板文件拷贝到客户端中。

在本例中，我们的`client1`设备需要它的证书和key,位于服务器：

- `/etc/openvpn/easy-rsa/keys/client1.crt`
- `/etc/openvpn/easy-rsa/keys/client1.key`

`ca.crt` 和`client.ovpn `文件对所有客户端都是一样的。也需要下载这两个文件：注意 `ca.crt` 文件位置和其它不同：

- `/etc/openvpn/easy-rsa/keys/client.ovpn`
- `/etc/openvpn/ca.crt`

下载工具可以随意选择。以scp为例：

```
scp root@your-server-ip:/etc/openvpn/easy-rsa/keys/client1.key Downloads/
```

## Creating a Unified OpenVPN Profile for Client Devices


修改client.ovpn

在文件顶部，找到下面片段：

```
# The hostname/IP and port of the server.
# You can have multiple remote entries
# to load balance between the servers.
remote my-server-1 1194
```

把 `my-server-1`修改为VPN的IP

接下来，取消 `user nobody` 和 `group nogroup`的注释。注意：如果是windows，可以跳过这一步。修改结果如下：

```
# Downgrade privileges after initialization (non-Windows only)
user nobody
group nogroup
```

取消注释下面片段的最后三行，这样我们就可以直接把证书和key的内容直接写进文件中。修改为：

```
# SSL/TLS parms.
# . . .
#ca ca.crt
#cert client.crt
#key client.key
```

将ca.crt client1.crt 和client,key 文件内容拷贝进 .ovpn 。使用XML-like 语法：

```
<ca>
(insert ca.crt here)
</ca>
<cert>
(insert client1.crt here)
</cert>
<key>
(insert client1.key here)
</key>
```

完成之后，应该类似下面：

```
<ca>
-----BEGIN CERTIFICATE-----
. . .
-----END CERTIFICATE-----
</ca>

<cert>
Certificate:
. . .
-----END CERTIFICATE-----
. . .
-----END CERTIFICATE-----
</cert>

<key>
-----BEGIN PRIVATE KEY-----
. . .
-----END PRIVATE KEY-----
</key>
```

client.crt 文件中有些额外的信息，把整个文件拷贝进去也OK。

## Step 5 - Installing the Client Profile


### windows
### OSX

## Step 6 - Testing Your VPN Connection























 



































原文地址：[https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-ubuntu-14-04](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-ubuntu-14-04)