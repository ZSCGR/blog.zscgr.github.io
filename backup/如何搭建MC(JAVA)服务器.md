# 前言
搭建MC服务器的教程已经有非常详细的了[MCSManager](https://github.com/MCSManager/MCSManager/blob/master/README_ZH.md#linux)，所以本文主要是方便我自己使用。
# 正文
默认环境为debian 12
## 一键安装命令(不是重点)
`sudo su -c "wget -qO- https://script.mcsmanager.com/setup_cn.sh | bash"`

- 脚本仅适用于 `Ubuntu/Centos/Debian/Archlinux`。
- 面板代码与运行环境自动安装在 `/opt/mcsmanager/` 目录下。
## 手动安装
- 若一键安装不起作用，则可以尝试此步骤手动安装。
```shell
# 切换到安装目录。如果不存在，请提前用'mkdir /opt/'创建它。
cd /opt/
# 下载运行时环境（Node.js）。如果你已经安装了Node.js 16+，请忽略此步骤。
wget https://nodejs.org/dist/v20.11.0/node-v20.11.0-linux-x64.tar.xz
# 解压档案
tar -xvf node-v20.11.0-linux-x64.tar.xz
# 添加程序到系统环境变量
ln -s /opt/node-v20.11.0-linux-x64/bin/node /usr/bin/node
ln -s /opt/node-v20.11.0-linux-x64/bin/npm /usr/bin/npm

# 准备好安装目录
mkdir /opt/mcsmanager/
cd /opt/mcsmanager/

# 下载MCSManager
wget https://github.com/MCSManager/MCSManager/releases/latest/download/mcsmanager_linux_release.tar.gz
tar -zxf mcsmanager_linux_release.tar.gz

# 安装依赖库
./install-dependency.sh

# 请打开两个终端或screen

# 先启动节点程序
./start-daemon.sh

# 启动网络服务(在第二个终端或screen)
./start-web.sh

# 为网络界面访问http://localhost:23333/
# 一般来说，网络应用会自动扫描并连接到本地守护进程。
``` 
**然后配置守护进程**
运行命令 nano /etc/systemd/system/mcsm-web.service 来编辑web面板的服务，输入下面的内容，然后按 Ctrl + O 再按回车来保存，接着按 Ctrl + X 来退出。
```
# /etc/systemd/system/mcsm-web.service
[Unit]
Description=MCSM 9 Web

[Service]
WorkingDirectory=/opt/mcsmanager/web
ExecStart=/usr/bin/node app.js
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

[Install]
WantedBy=multi-user.target
```
运行命令 nano /etc/systemd/system/mcsm-daemon.service 来编辑web面板的服务，输入下面的内容，然后按 Ctrl + O 再按回车来保存，接着按 Ctrl + X 来退出。
```
# /etc/systemd/system/mcsm-daemon.service
[Unit]
Description=MCSM 9 Daemon

[Service]
WorkingDirectory=/opt/mcsmanager/daemon
ExecStart=/usr/bin/node app.js
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

[Install]
WantedBy=multi-user.target
```
## 安装后使用方法
```
systemctl start mcsm-{web,daemon} # 开启面板
systemctl stop mcsm-{web,daemon}  # 关闭面板
systemctl daemon-reload # 重新载入变更
systemctl enable mcsm-{daemon,web} # 设置开机自启动

# 显示运行状况和日志
systemctl status mcsm-web.service
systemctl status mcsm-daemon.service -l

```
## 安装服务器环境
**在安装MC服务端之前建议先安装JAVA环境**
各版本MC与JAVA对应关系如下
<table>
  <tr>
    <th>Minecraft版本号（Java版）</th>
    <th>最低启动版本</th>
    <th>推荐安装的 Java 版本</th>
    <th>下载路径</th>
  </tr>
  <tr>
    <td>1.7-</td>
    <td>8</td>
    <td>8</td>
    <td><a href="https://www.oracle.com/in/java/technologies/javase/javase8-archive-downloads.html">JAVA 8</a></td>
  </tr>
  <tr>
    <td>1.8+ ~ 1.15+</td>
    <td>8</td>
    <td>11</td>
    <td><a href="https://www.oracle.com/java/technologies/javase/jdk11-archive-downloads.html">JAVA11</a></td>
  </tr>
  <tr>
    <td>1.16+</td>
    <td>11</td>
    <td>16</td>
    <td><a href="https://www.oracle.com/java/technologies/javase/jdk16-archive-downloads.html">JAVA16</a></td>
  </tr>
  <tr>
    <td>1.17+ ~ 1.19+</td>
    <td>13</td>
    <td>17</td>
    <td><a href="https://www.oracle.com/java/technologies/javase/jdk17-archive-downloads.html">JAVA17</a></td>
  </tr>
  <tr>
    <td>1.20 系列</td>
    <td>17</td>
    <td>21</td>
    <td rowspan="3"><a href="https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html">JAVA 21</a></td>
  </tr>
  <tr>
    <td>24W14A（快照）及以后版本</td>
    <td>21</td>
    <td>暂无</td>
  </tr>
  <tr>
    <td>1.21（即将到来）</td>
    <td>21</td>
    <td>暂无</td>
  </tr>
</table>

## 安装原版服务器
### 方法一
最直接方便的方法就是安装完面板后登录，点击左侧的快速开始 -> 创建一个 Minecraft 服务器 -> 一键开服，缺点是只有1.17.1-1.19.2版本。
![版本](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/37fc31f8-6511-4292-9e6b-e8a7fc8f3b9e)
### 方法二
可以自行去找服务端，下面以[paper](https://papermc.io/downloads/all)端为例，下载自己需要的版本，然后按照方法一的流程上传，点击左侧的快速开始 ->创建一个 Minecraft 服务器 ->普通流程创建服务器 -> Java 版 Minecraft 游戏服务端 -> 上传单个服务端软件（推荐，将下载的server.jar上传上去。
![上传](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/fe1261a3-f639-4c68-a138-4a270d54a800)
然后进入这个页面，点击控制台。
![参数](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/7999fa8e-0dd1-45d0-84a2-450a5b243983)
接着点**开启实例**即可，等待安装完成，国内服务器因为国内特殊的网络环境原因有下载失败的概率。
![控制台](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/0e282ee6-e448-4018-b753-c02518d00cc8)
## 安装forge端服务器
首先去[forge](https://files.minecraftforge.net/net/minecraftforge/forge/)官网下载所需的版本，以MC1.18.2，forge40.2.0为例，上传到和server.jar同级的目录里，然后运行
`java -jar forge-1.18.2-40.2.0-installer.jar -installServer'
运行完成后会得到一个run.sh文件，修改里面参数
```java
java @user_jvm_args.txt @libraries/net/minecraftforge/forge/1.18.2-40.2.0/unix_args.txt nogui "$@"
``` 
![run](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/1dc47dbe-8c3e-4eaa-a17a-af876afc051e)
然后进入高级实例设置，修改启动命令为`./run.sh`或者为`sh run.sh`或者`bash run.sh`等等
![参数-2](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/74e37a78-9432-459c-88d9-32bae0ae0d60)
## 后续问题
### 找不到java路径怎么办？
答：把教程中使用到Java的地方路径全部替换成绝对路径。
### 提示需要同意EULA协议
![eula](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/c922dd69-3423-423a-89f7-49c12b1f1dd9)
答：修改eula.txt为true。
![MC](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/42661a86-11b4-4153-a408-ab8aa5fbe6bb)
或者从此进入服务端配置文件
![aEULA](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/f1669287-fbdb-4534-9849-804c7eb5be43)
然后点击eula.txt.
![aEULA-2](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/e9d9f806-7e2d-42fe-aaf1-4b5b48be2a2d)
最后改为"是"
![aEULA-3](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/ec78e219-8a92-4c19-929e-351ad3611d25)
