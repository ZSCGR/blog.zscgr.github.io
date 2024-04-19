# 介绍
cloudflare R2是一个文件储存系统，配合Cloudflare Workers可以实现这样一个网盘系统。
# 起因
在[LINUX DO](https://linux.do)刷帖的时候知道了图床这个东西，但一直都都没想好怎么用，后面想了想，自己部署网站很多时候都是没有图标或者是默认的图标，实在不美观，于是打算开始建图床，一开始刷到的一些图床项目是基于Telegraph的，我自己觉得不太好用，直到看到了这个[图床](https://github.com/ljxi/Cloudflare-R2-oss)项目，觉得基本能满足自己的需求，不仅能设置管理员权限，还能设置游客的权限，能直接在网页上预览图片，非常的不错。免费的量也足够我自己一个人使用了。
![R2](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/e9825adf-f990-48bb-92dd-e63d76ce9bb3)
# 部署
首先fork这个[图床](https://github.com/ljxi/Cloudflare-R2-oss)，然后去cloudflare创建一个R2存储桶。
![R2-create](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/66bda92b-461e-4ba8-9c76-0b99527cf974)
然后前往Cloudflare Pages新建一个站点，选择连接到Git

![page-1](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/0de6e171-2e5a-4753-be6d-5833a72b60a8)

![page-2](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/3d327fff-57e1-4ff4-ae91-0d18e61c78cc)

接着连接到刚刚fork的github仓库，项目名称可以修改，其他选项保持默认不动。

![部署页面](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/a17e4145-9100-4b25-9b88-5f0f0589a412)

然后展开环境变量，按需填写。

![环境变量](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/77662304-995e-4c71-8deb-017d8b5d3656)

|  **变量名称**   | **值**  |
|  ----  | ----  |
| PUBURL  | 公共存储桶URL |
| GUEST  | public/ |
| admin:123456  | * |
| user1:123456  | user1/,userPublic/ |

其中，GUEST代表游客的允许写入目录
管理员则以`账号:密码`的形式设置，值代表其允许写入的目录，用,隔开，**请勿在前后加逗号**，否则会授予**所有目录的写入权限。**
设置好后点击**开始部署**
然后前往Pages->cloudflare-r2-oss->设置->函数->R2 存储桶绑定,绑定R2存储桶,变量名称`BUCKET`

![绑定R2](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/3564f2e0-f67c-4f77-98f8-df36618ec97e)

最后在部署页面重新部署即可。

![rebuild](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/6c2ed7a7-30a6-4900-9d72-8b8c4611c137)