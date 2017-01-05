---
layout: post
title:  "为群晖nas docker套件配置镜像加速下载"
date:   2017-01-05 20:27:00 +0800
categories: 群晖 docker
---



自从去年618在京东买的两块4T红盘因为坏道更换两次，重装了NAS统自后，下载Docker镜像一直不成功，查询registry经常失败之后，就一直在想办法解决这个问题：

- 给电信客服打电话要独立ip
- 从其他虚拟机docker save镜像上传到NAS并docker import
- 网上搜索其他人遇到的同类问题，有人反映synology对docker套件不提供支持，那只能自己想办法解决了
- 有人提到可能是网卡[MTU](https://zh.wikipedia.org/zh-hans/最大传输单元)设置问题，按照网上指导修改MTU为1500仍然没有解决问题



最后想到有可能只是网络问题，可以为docker设置registry mirror加速镜像下载，或许可以解决这个问题：

- 登录[阿里云](https://cr.console.aliyun.com)(用淘宝帐号登录，没人没有淘宝帐号吧)，点击左侧菜单`加速器`，在页面上方找到你的专属加速器地址，例如`https://xxxxxxxx.mirror.aliyuncs.com`
- 开启NAS SSH（开启位置在`控制面板>终端机和SNMP>启动SSH功能`）
- 登录后台，找到`/etc/profile`文件增加如下两行，`registry-mirror`是你的专属加速器地址

```
DOCKER_OPTS="$DOCKER_OPTS --registry-mirror=https://xxxxxxxx.mirror.aliyuncs.com"
export DOCKER_OPTS
```

- 重启docker，重启NAS，执行`ps -ef|grep docker`，发现docker daemon启动参数没有变化
- 找到docker启动脚本，位置在`/var/packages/Docker/scripts/start-stop-status`找，找到`#start docker`这一行，将接下来的一行由

```
"${DockerBin}" daemon --ipv6=true &
```

修改为

```
"${DockerBin}" $DOCKER_OPTS --ipv6=true &
```

- 重启NAS之后继续执行`ps -ef|grep docker`发现`registry-mirror`依旧没有生效（不知为何没有执行`/etc/profile`文件，请高手指点）
- 环境变量不行，那就直接加参数吧，还是上边那个文件，将原来那一行修改为：

```
"${DockerBin}" --registry-mirror=https://xxxxxxxx.mirror.aliyuncs.com daemon --ipv6=true &
```

- 重启docker之后，`registry-mirror`参数生效了，实际测试发现下载镜像加速效果还不错，这里给阿里赞一个



赶紧pull几个一直想搞的镜像run起来，目前在用的镜像有：

- nofish/zeronet：[zeronet](https://zh.wikipedia.org/zh-hans/ZeroNet)一个基于p2p的网络，建站再也不用找空间了，而且可以反封锁
- wan0eve/ss-local：鼎鼎大名的shadowsocks客户端，实现socks代理
- janeczku/calibre-web：电子书管理软件，calibre的web版本



如果你也遇到这个问题，赶紧去试试吧，即使没有这个问题，也可以大大加速docker镜像的下载