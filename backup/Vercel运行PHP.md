# 起因
在B站看到一个点击按钮计数的项目[原神主题网页](https://gitee.com/KleeGitee/Klee),由于不会构造后端，当时也不太会代码，所以就用了php构造后端来连接数据库，用ChatGPT写了请求和修改的逻辑（面向ChatGPT编程 :smile:），自己部署的是我自己修复了一个原项目bug的版本。
项目地址：[原神，启动!](https://ys.chgr.cc)
# 正篇
首先要用到的是[vercel-community/php](https://github.com/vercel-community/php)这个仓库，这个仓库提供了在Vercel上运行PHP的方法，只要按照这个仓库里的说明，修改自己的php项目中的结构，就可以在Vercel上运行了。而[php样例](https://github.com/juicyfx/vercel-examples) 这个仓库提供了很多类型的可供部署的php项目。
# 遇到的问题
第一次在Vercel上部署项目的时候会失败，这是因为vercel默认的node.js版本是20.x，而vercel-php使用的node.js版本是18.x。
![搜狗截图20240408141855](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/56fdab32-58cd-43a7-82ee-c96e2caffdaf)

**但是没关系，我们回到创建项目的界面，进入部署失败的项目，在设置界面将node.js的版本改成18.x。**

![搜狗截图20240408174619](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/91e4456f-f7c1-4a6f-9c79-33adb553c40d)

**然后进入部署页面重新部署**
![搜狗截图20240408142256](https://github.com/ZSCGR/blog.zscgr.github.io/assets/75410405/02f8457d-0159-4236-ab48-23528e8c868d)

最后php项目就可以成功运行在Vercel上了。以及下面是我托管在Vercel上的一个php网站，因为主要是演示用，所以我就用`Carbon`简单写了一下获取当前的时间。
项目地址：[获取当前时间 UTC+8](https://php.chgr.cc)