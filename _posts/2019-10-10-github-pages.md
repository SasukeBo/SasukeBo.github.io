---
title:  "使用Github Pages搭建个人博客"
date:   2019-10-10 18:53:00 +0800
categories: Github
header:
  overlay_image: /assets/images/banners/github-pages.png
  teaser: /assets/images/banners/github-pages.png
comments: true
---

使用`Jekyll`和`Minimal Mistakes`在`Github Pages`上搭建个人博客。

## Github Pages

搭建简单的github个人主页十分简单。只需要几步:
- 创建一个仓库，用于存放个人博客页面内容。命名按照格式 *username.github.io*，`username`需要和你的github用户名一致。
![create repository](/assets/images/github-pages/user-repo@2x.png)
- 克隆你的博客仓库到本地。
  ```sh
  ~ $ git clone https://github.com/username/username.github.io
  ```
- 添加首页文件
  ```sh
  ~ $ cd username.github.io
  ~ $ echo "Hello World" > index.html
  ```
- 提交你的博客仓库
  ```sh
  ~ $ git add .
  ~ $ git commit -m "Initial commit"
  ~ $ git push -u origin master
  ```

最后访问你的个人主页：*[https://username.github.io.](https://username.github.io.)*

**Note:** 当然，这样简单的静态页面展示不能满足你的需求，所以github提供了对[Jekyll](https://jekyllrb.com/)框架的支持，让你能够快速搭建相对复杂的web项目，用于个人博客已经绰绰有余，并且文章支持[markdown](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)格式。
{: .notice--info}

## 使用Jekyll搭建你的个人主页

### Jekyll简介

Jekyll是一个静态网站生成工具，github提供了服务端build支持，你只需要上传代码，github服务器就可以使用Jekyll编译并生成静态网站。
Jekyll支持`Markdown`和`Liquid`语法。使用`Markdown`和`HTML`文件生成静态页面网站。

### 安装Jekyll

Jekyll是`Ruby`项目，所以你需要安装完整的`Ruby`运行环境，其中`Ruby`版本不低于`2.4.0`，安装教程[Installation](https://jekyllrb.com/docs/installation/)。

### 创建Jekyll项目

安装完成Jekyll后，可以使用`jekyll new`命令创建一个新项目。
```sh
$ bundle exec jekyll new .
```

### 测试你的项目

可以在本地运行该项目：
```sh
$ bundle install
$ bundle exec jekyll serve
```

本地服务启动后浏览器访问[http://localhost:4000](http://localhost:4000)即可看到博客首页。

**Note:** 本地服务可以自动编译修改或新增的文件，但修改`_config.yml`文件是需要重启本地服务的。
{: .notice--danger}

### Jekyll Theme

有了博客，还会想到去美化博客样式，以及增加一些拓展功能。Jekyll提供主题功能。
Github Pages提供了一些[主题](https://pages.github.com/themes/)支持。使用Github Pages提供的主题只需要在`_config.yml`文件中写入`theme: THEME_NAME`。Jekyll支持的主题更加丰富，可以从下列站点挑选你心仪的Theme：
- [jamstackthemes.dev](https://jamstackthemes.dev/ssg/jekyll/)
- [jekyllthemes.org](http://jekyllthemes.org/)
- [jekyllthemes.io](https://jekyllthemes.io/)

本人使用的主题是[Minimal Mistakes Jekyll theme](https://mmistakes.github.io/minimal-mistakes/)。

## Minimal Mistakes Jekyll Theme

个人比较喜欢这个风格的主题，Github stars 5.7k，fork 9.6k，还是比较火的。可能有五六千人用这个主题搭建了个人博客。
这个[主题](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/)文档也比较详细，就不赘述了，主要记录一些我在部署过程中遇到的问题。

### Deploy Problems

- 由于不是Github Pages支持的主题，使用该主题需要用`remote_theme: "mmistakes/minimal-mistakes@4.17.0"`，当然本地测试的时候推荐使用`theme: minimal-mistakes-jekyll`，需要使用`gem`下载该主题，添加到Gemfile中。
- 覆盖主题
  ```
  ├── _includes
  ├── _layouts
  ├── _sass
  ├── assets
  |  ├── css
  |  ├── fonts
  |  └── js
  ```
  可以通过在项目中编辑这些目录下的文件来覆盖主题。
- 建议在遇到一些显示问题时，去gem包中查看主题模板，检查模板中引用的参数是否在配置文件中正确配置了。
- 路由实际上对应的就是build生成的`_site`目录下的文件。如果博客中路由404可以对照`_site`目录结构检查链接配置。
- `site locale`中文设置为`zh`，貌似`zh-CN`不工作，但是看源码这个就是`zh`的别名。

### Comments

**给自己的博客添加评论功能: [Comments](https://mmistakes.github.io/minimal-mistakes/docs/configuration/#comments)**

`provider`推荐使用[staticman_v2](https://staticman.net/docs/index.html)，貌似别的都收费。

`staticman_v2`的工作原理是，提供一个API服务器接收评论，之后以`staticman`的身份向你的Github博客项目指定分支提交`pull request`。

评论存储到项目的`_data/comments`目录下，根据文章分类创建下一级文件夹。

当然`staticman`需要你提供项目的读写权限，官网文档有点过时，提供的方法是添加`Collaborators`，如果你去搜索会发现`staticmanpp`不是github user，因此不能通过添加`Collaborators`来授权，根据[issue#243](https://github.com/eduardoboucas/staticman/issues/243#issuecomment-453754860)给出的方案，通过在项目中安装`staticman app`来授权。

评论form的action链接亲测可用的是：[https://dev.staticman.net/v3/entry/github/USERNAME/REPOSITORY/BRANCH](https://dev.staticman.net/v3/entry/github/[USERNAME]/[REPOSITORY]/[BRANCH])，其他的不清楚是否可用。

另外需要在项目中设置[Webhooks](https://staticman.net/docs/webhooks)，[https://github.com/USERNAME/REPOSITORY/settings/hooks](https://github.com/USERNAME/REPOSITORY/settings/hooks)。按照教程来吧，`Payload URL`我设置的是和 form action 对应的URL。其余方式没有验证是否可行。

另外有一个疑问，博客仓库设置为`private`是否会影响站点发布，有时间再研究研究。
