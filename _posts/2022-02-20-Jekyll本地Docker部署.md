---
layout: post
title: "Jekyll本地Docker部署"
categories: Docker
tags: Docker Jekyll
author: August
mathjax: true
typora-root-url: ..
---

* content
{:toc}

本文介绍 Jekyll  本地 Docker 部署。



# Jekyll  本地 Docker 部署

由于我的**Github Pages**依赖于jekyll，每次有较复杂的排版时总是会出现莫名其妙的问题（可能是jekyll解析的Markdown格式有出入），所以需要配置一个本地的预览环境。但是不想主机OS上安装过多的定制化功能，技术路线选择了**Docker**容器。



## Jekyll

Jekyll 是一个简单的博客形态的静态站点生产机器。它有一个模版目录，其中包含原始文本格式的文档，通过一个转换器（如 Markdown）和我们的 Liquid 渲染器转化成一个完整的可发布的静态网站，你可以发布在任何你喜爱的服务器上。Jekyll 也可以运行在 GitHub Page 上，也就是说，你可以使用 GitHub 的服务来搭建你的项目页面、博客或者网站，而且是完全免费的。

### 目录结构

| 文件 / 目录                                          | 描述                                                         |
| ---------------------------------------------------- | ------------------------------------------------------------ |
| `_config.yml`                                        | 保存[配置](http://jekyllcn.com/docs/configuration/)数据。很多配置选项都可以直接在命令行中进行设置，但是如果你把那些配置写在这儿，你就不用非要去记住那些命令了。 |
| `_drafts`                                            | drafts（草稿）是未发布的文章。这些文件的格式中都没有 `title.MARKUP` 数据。学习如何 [使用草稿](http://jekyllcn.com/docs/drafts/). |
| `_includes`                                          | 你可以加载这些包含部分到你的布局或者文章中以方便重用。可以用这个标签 `{% include file.ext %}` 来把文件 `_includes/file.ext` 包含进来。 |
| `_layouts`                                           | layouts（布局）是包裹在文章外部的模板。布局可以在 [YAML 头信息](http://jekyllcn.com/docs/frontmatter/)中根据不同文章进行选择。 这将在下一个部分进行介绍。标签 `{{ content }}` 可以将content插入页面中。 |
| `_posts`                                             | 这里放的就是你的文章了。文件格式很重要，必须要符合: `YEAR-MONTH-DAY-title.MARKUP`。 [永久链接](http://jekyllcn.com/docs/permalinks/) 可以在文章中自己定制，但是数据和标记语言都是根据文件名来确定的。 |
| `_data`                                              | 格式化好的网站数据应放在这里。jekyll 的引擎会自动加载在该目录下所有的 yaml 文件（后缀是 `.yml`, `.yaml`, `.json` 或者 `.csv` ）。这些文件可以经由 ｀site.data｀ 访问。如果有一个 `members.yml` 文件在该目录下，你就可以通过 `site.data.members` 获取该文件的内容。 |
| `_site`                                              | 一旦 Jekyll 完成转换，就会将生成的页面放在这里（默认）。最好将这个目录放进你的 `.gitignore` 文件中。 |
| `.jekyll-metadata`                                   | 该文件帮助 Jekyll 跟踪哪些文件从上次建立站点开始到现在没有被修改，哪些文件需要在下一次站点建立时重新生成。该文件不会被包含在生成的站点中。将它加入到你的 `.gitignore` 文件可能是一个好注意。 |
| `index.html` and other HTML, Markdown, Textile files | 如果这些文件中包含 [YAML 头信息](http://jekyllcn.com/docs/frontmatter/) 部分，Jekyll 就会自动将它们进行转换。当然，其他的如 `.html`, `.markdown`, `.md`, 或者 `.textile` 等在你的站点根目录下或者不是以上提到的目录中的文件也会被转换。 |
| Other Files/Folders                                  | 其他一些未被提及的目录和文件如 `css` 还有 `images` 文件夹， `favicon.ico` 等文件都将被完全拷贝到生成的 site 中。这里有一些[使用 Jekyll 的站点](http://jekyllcn.com/docs/sites/)，如果你感兴趣就来看看吧。 |



## Docker

**Docker容器**与虚拟机类似，但二者在原理上不同。容器是将[操作系统层虚拟化](https://zh.wikipedia.org/wiki/作業系統層虛擬化)，虚拟机则是虚拟化硬件，因此容器更具有便携性、更能高效地利用服务器。 容器更多的用于表示软件的一个标准化单元。由于容器的标准化，因此它可以无视基础设施（Infrastructure）的差异，部署到任何一个地方。另外，Docker也为容器提供更强的业界的隔离兼容。



## 部署

### 下载 Docker

官网: [https://www.docker.com/](https://www.docker.com/)

Windows 直接下载桌面版安装即可，会重启电脑并自动将命令添加在环境变量中。

### 下载 Jekyll

```dockerfile
docker pull jekyll/jekyll
```

### 使用 Jekyll

#### 下载 Jekyll 博客模板

在 GitHub 上下载一个模板即可。

```dockerfile
git clone git@github.com:august295/august295.github.io.git
```

#### 本地部署

```dockerfile
docker run -it --name my-jekyll -p 4000:4000 -v D:\\Github\\august295.github.io:/srv/jekyll jekyll/jekyll jekyll serve
```

- my-jekyll: 创建镜像名称
- `4000:4000` 容器的端口映射
- D:\\Github\\august295.github.io: Windows 本地路径
- /srv/jekyll: 映射路径
- jekyll/jekyll: 指定上述下载的 Jekyll
- jekyll serve: Jekyll 启动服务

![](/media/image/2022-02-20-Jekyll本地Docker部署/docker-jekyll-server.png)

#### 其他命令

```dockerfile
# 
```

