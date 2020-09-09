# 在 openEuler 上安装桌面环境

openEuler 是华为自主研发的，主要用于其鲲鹏服务器上的一款基于 CentOS 修改而来的 Linux 发行版。

> 有句名言要牢记，国外一开源，国内就自主。

本来作为已经叛逃到 macOS 的老 Linux 玩家，我个人对于常用的 Ubuntu 之外的发行版是没有过多的兴趣的。不过因为某些恶心人的课程的原因，我不得不需要在 openEuler 上运行基于图形界面的  `qemu` 以展示我们的操作系统。但是作为一款主要面向服务器的操作系统，华为官方的[安装指南](https://openeuler.org/zh/docs/20.03_LTS/docs/Quickstart/quick-start.html)并未给出安装桌面环境的步骤，我不得不自己动手，丰衣足食。以下是安装步骤。

### 配置源

是的，你没有看错，在 0202 年，除了 LFS 这种适合高阶选手定制的发行版外，还有发行版敢不内置一两个官方源。华为倒是提供了官方源的[配置方式](https://openeuler.org/zh/docs/20.03_LTS/docs/Administration/%E4%BD%BF%E7%94%A8DNF%E7%AE%A1%E7%90%86%E8%BD%AF%E4%BB%B6%E5%8C%85.html)，不过内置源这种举手之劳就可以大幅改善用户体验的事情他们居然不做，还是很令人震惊的。

这一步我们要做的就是

```bash
sudo vim /etc/yum.repos.d/openEuler_x86_64.repo
```

打开文件，并将这部分文件复制进去，当然如果在虚拟机下此时可能无法复制，就只好麻烦多敲几下键盘手动输入进去了。

```apacheconf
[osrepo]
name=osrepo
baseurl=https://repo.openeuler.org/openEuler-20.03-LTS/OS/x86_64/
enabled=1
gpgcheck=1
gpgkey=https://repo.openeuler.org/openEuler-20.03-LTS/OS/x86_64/RPM-GPG-KEY-openEuler
```

#### 使用清华源

经过舍友实测，openEuler 的官方源速度不是十分理想，如果想要一个可以接受的速度的话，换源是必需的。以下是使用清华源的 `openEuler_x86_64.repo`，直接复制到文件里即可。

```apacheconf
[osrepo]
name=osrepo
baseurl=https://mirrors.tuna.tsinghua.edu.cn/openeuler/openEuler-20.03-LTS/OS/x86_64/
enabled=1
gpgcheck=1
gpgkey=https://mirrors.tuna.tsinghua.edu.cn/openeuler/openEuler-20.03-LTS/OS/x86_64/RPM-GPG-KEY-openEuler
```

### 安装 GNOME

gnome 是很常用的桌面环境，以下为安装指令

```bash
sudo dnf install gnome-shell gdm gnome-session
```

等待大约三分钟，就安装完毕了。

### 设置 gdm 开机自启

```bash
sudo systemctl enable gdm.service
sudo systemctl set-default graphical.target
```

至此，通常情况下的 gnome 桌面环境已经安装完毕。但是如果你此时贸然重启，会遇到 gdm 无法登陆的问题。

### 补全丢失文件

在上一节已经说过了，在 openEuler 直接未经修改直接启动 gdm 会遇到无法登陆的问题，具体表现为登陆了过几秒就直接回到了 gdm 的登陆界面。这个问题是由于 openEuler 的 `gdm` 的配置文件不全导致的。具体来说，是 `/etc/gdm/Xsession` 指向的 `/etc/X11/Xsession` 不存在。不管怎么说，出现了这种问题都说明华为居然敢把未经测试的软件直接上马官方软件源，这种行为在任何标榜自己安全、稳定的 Linux 发行版上出现都是不可理喻的。希望有选择 openEuler 作为自己生产用操作系统的想法的人看到本文后立刻打消想法。今天他华为敢未经测试就把配置文件不全的包直接丢给客户用，明天 [Bumblebee 事件](https://lantian.pub/article/forward/foolish-code-typo.lantian/)再次出现时候被删库的就是你们。

要解决这个问题也简单，我们首先切换到 `root` 用户，然后从网络上下载并替换该文件即可，步骤如下：

```bash
cd /tmp
wget https://gitee.com/name1e5s/xsession/raw/master/Xsession
mv Xsession /etc/gdm/
chmod 0777 /etc/gdm/Xsession
```

至此，openEuler 的桌面环境已经完全可用，如果你注意到了没有终端，直接 `dnf install gnome-terminal` 即可安装。以下是安装好后的背景

![](https://raw.githubusercontent.com/name1e5s/article/master/pic/openEuler.png)

至于那位不知道 openEuler 和别的 Linux 发行版区别的老师怎么分辨这个国产 OS 之光和其他使用 GNOME 作为桌面环境的发行版，我暂且蒙在鼓里。

### openEuler 壁纸下载

[点此即可](https://raw.githubusercontent.com/name1e5s/article/master/pic/openEuler.zip)


