# 前言
前一篇文章我们讲了如何搭建MC服务器，[#5](https://blog.chgr.cc/post/5.html)，现在我们来讲如何正式开服。本MC服务器搭建于华为鸿蒙4的termux中的debian 12中（旧手机废物利用 😄 ）。
# 正文
## 安装termux
首先在手机上下载好termux，进入设置 -> 应用和服务 -> 应用启动管理 中把termux设置为手动管理，主要是允许后台活动这个选项。
然后进入termux，运行
```shell
bash -c "$(curl -fsSL https://gitee.com/mo2/linux/raw/2/2)"
```
不出意外的话手机应该就可以弹出gui界面了，选择第一个proot（手机非root）。
![proot](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/0a3240b7-ef27-48cf-acca-795f0a295983)
直接回车默认arm64架构
![arm64](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/3499ffaf-8574-46ed-bfcf-6f98e5b73e61)
我选择的是第三个，debian系统
![debian](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/e7cf809f-0fbe-409f-8976-56cf8f736ac9)
我选择debian 12系统
![debian12](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/95576a8d-415a-452b-a6a3-e13bdffeb6fc)
启动系统，然后进行配置（配置linux就不详细讲了）
**然后`sudo -s`进入root**
![启动](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/38286efa-d4e4-45ab-bb1e-184c990fe2eb)
然后就可以根据上一篇教程安装[MCSManager](https://github.com/MCSManager/MCSManager/blob/master/README_ZH.md#linux)了
## 内网穿透
内网穿透就不必自说了，这里首选 OpenFrp。
### [OpenFrp](https://www.openfrp.net)
注册登录OpenFrp后，点击左边侧边栏软件下载,，点击复制链接
![openfrp](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/13a60dac-0baf-49a6-b3e7-d34bcd863bbb)
在下载链接前加上`wget`命令，即
```sh
wget https://o.of.gs/client/OpenFRP_0.57.0_e511492b_20240423/frpc_linux_arm64.tar.gz
```
解压
```
tar -zxvf frpc_linux_arm64.tar.gz
```
重命名
```
mv frpc_linux_arm64 frpc
```
赋予执行权限
```
chmod +x frpc
```
测试运行
```
./frpc
```
没问题好话回到官网，创建隧道，隧道类型`TCP`，本地地址`127.0.0.1`，本地端口`25565`，MC默认端口为25565，修改过的话就填修改后的，远程端口随机。
创建完成后点击管理隧道，然后就可以看到隧道了，点击最右边的小扳手，获取配置文件/启动命令，点击复制，粘贴到debian系统中，别急着运行，用nohup后台运行
```
nohup ./frpc -u 用户秘钥 -p 端口ID-名称 > /path/name.log 2>&1 &
```
- `nohup`是常用的后台运行命令，这样在你关闭ssh窗口后程序仍然会在后台运行，否则就会随着你ssh窗口关闭而结束。
- `/path`是你要储存日志的目录。
- `name.log`是日志的名字，可以随便去，后缀也不一定得是`.log`。
然后就可以在MC多人游戏中根据OpenFrp提供的域名/ip+远程端口进行连接了。
### [樱花穿透](https://www.natfrp.com)
流程和OpenFrp差不多，点击上边状态栏中的服务中的软件下载 -> Linux / FreeBSD 启动器 -> Linux AArch64 / ARM64 -> 复制链接
![sakura](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/56494e8f-5125-40f4-a1ee-c18f96dc0edc)
同样在前面加上`wget`命令
```
wget https://nya.globalslb.net/natfrp/client/launcher-unix/3.1.0/natfrp-service_linux_arm64.tar.zst
```
解压
```
tar -I zstd -xvf natfrp-service_linux_arm64.tar.zst
```
赋予权限
```
chmod +x frpc
```
后续运行、创建隧道基本相同，复制启动参数的时候略有不同，sakura frp是-f参数，即
```
nohup ./frpc -f 用户秘钥:隧道ID  > /path/name.log 2>&1 &
```
其他同类型的就不多说了
### [CLOUDFLARE](https://www.cloudflare.com)
这个比较难弄，需要绑定卡或者paypal账号，国内建议后者，上上篇文章用到过#4
首先进入 Zero Trust 后台
![index](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/e82fe509-5c0a-468f-9095-06b300153a04)
选择「Network」-> 「Tunnel」-> 「Create a Tunnel」
![tunnel](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/bd880c4e-5cee-4367-bec3-0e5b8fc01836)
选择「Cloudflared」
![choose](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/2690f500-927d-494a-877a-2fbd9fddf962)
填入该隧道的注释名称
![MC](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/0e281e18-d22d-4d7c-9d34-9f8a9daa9597)
点击方框内的内容, 复制安装命令
![cloudflared](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/bf8b5805-2903-480f-948f-cacb1f00c545)
然后点击配置进入下面的页面（用的之前的图，懒得换了）
![public](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/f53e5df5-71dc-4db9-9d78-92d6ddaaade6)
然后在下一个页面中填入如图所示内容
![mc-yuming](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/1a59bddf-89ce-4637-bdb2-937c5fb7a1ce)
主要是得有一个托管于cloudflare的域名（上上篇好像忘说了,诶嘿~）
然后就能在mc中通过自己的域名进行访问了。