---
title: "使用Hugo创建个人博客"
date: 2019-12-14T18:55:11+08:00
draft: false
---

# 使用Hugo搭建个人博客

## 一、下载Hugo 
### Mac
-   Mac下直接使用Homebrew安装即可(需要命令行科学上网)
```bash
brew install hugo
```
-   如果没有配置科学上网,可以去[Hugo releases](https://github.com/gohugoio/hugo/releases)下载
-   查看是否安装成功
```bash
hugo version
# 输出版本即为下载成功
```

### Windows
-   下载
    -   去[Hugo releases](https://github.com/gohugoio/hugo/releases)下载
-   解压
    -   后将hugo.exe放入你喜欢的目录中，比如`D:\Software\hugo` 
-   配置环境变量
    -   将hugo.exe所在文件夹添加到PATH。即`D:\Software\hugo` 

### 安装时的问题
#### 运行Hugo,`powershell`自动关闭
-   排查发现是电脑管家自动阻止了`Hugo`命令的运行,重启电脑就好啦
### 

## 二、创建一个 new Site
在任意一个目录下创建即可
### 官方推荐
```bash
hugo new site quickstart
```
### 使用github pages搭建
将new site名称改为xxx.github.io-creater，xxx 是你的github名称。注意**将大写字母改为小写**。
比如：Maricaya.github.io改为maricaya.github.io

```bash
hugo new site xxx.github.io-creater
```

## 三、添加一个主题
可以在[themes.gohugo.io](https://themes.gohugo.io/)中找到你喜欢的主题,我使用了[Ananke theme](https://themes.gohugo.io/gohugo-theme-ananke/)

```bash
# 切换目录
cd xxx.github.io-creater
# 初始化
git init

git submodule add https://github.com/budparr/gohugo-theme-ananke.git themes/ananke

# 在 config.toml中添加配置theme = "ananke"
echo 'theme = "ananke"' >> config.toml
```

## 四、开始写第一篇博客吧
```bash
hugo new posts/我的第一篇博客.md
```
打开`xxx.github.io-creater` 文件夹，`content/posts/我的第一篇博客.md` 就是第一篇博客的位置。
这是文章的初始化内容:
```md
---
title: "我的第一篇博客"
date: 2019-12-14T18:55:11+08:00
draft: true
---
```
**draft草稿状态，博客完成后，修改为false**

## 五、启动Hugo服务器
```bash
hugo server -D
```
打开http://localhost:1313 , 这就是你的博客啦

## 六、将博客改为中文
找到`config.toml`
```bash
baseURL = "http://example.org/"
# baseURL修改为`https://maricaya.github.io/`
languageCode = "en-us"
# languageCode="zh-Hans" 修改为中文简体
title = "My New Hugo Site"
theme = "ananke"
```

## 七、建立静态页面
这一步很简单，新开一个shell窗口，运行
```bash
hugo -D
```
创建了新的目录`public`

## 八、将博客搭建到Github Pages上
-   新建`.gitignore`，写入`/public/`，在git仓库中不上传public文件夹。`public`需要自成一个仓库
```bash
cd public 

git init

git add .
git commit -m "第一次部署"
```
-   在github中新建一个仓库，仓库名称为`xxx.github.io`。
    -   注意：xxx为你的github用户名，如果有大写字母必须改为小写。
    -   比如：我的用户名为Maricaya，仓库名为`maricaya.github.io`
    - 上传代码即可
    -  打开https:xx.github.io，即可查看你的博客啦
