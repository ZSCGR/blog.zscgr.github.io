# 起因
白嫖hax的ipv6小鸡的时候能通过ssh正常连接上主机，但是在进行`sudo apt update && sudo apt upgrade`命令更新源的时候显示无法解析到官方的apt源仓库，然后心血来潮尝试ping了一下外网，发现ping也无法解析域名，于是写下这篇文章记录解决的方法。
# 网络检查
首先可以使用`ping`命令测试一下网络是否正常，能否ping通外网。下面ping一下谷歌：
```
ping www.google.com
```
如果显示无法解析域名，则考虑DNS是否存在问题，可以打开`/etc/resolv.conf`这个文件查看里面的内容，添加或修改DNS。下面为添加的格式(均为谷歌的NDS)。

**ipv4：**
```
servername 8.8.8.8
servername 8.8.4.4
```
**ipv6：**
```
servername 8.8.8.8
servername 8.8.4.4
```
修改完成后理论上就可以正常上网了。