---
layout: post
title: "Jekyll本地Docker部署"
categories: Docker
tags: Docker
author: August
typora-root-url: ..
---

* content
{:toc}

本文介绍 `Jekyll` 本地 `Docker` 部署。



# Jekyll 本地 Docker 部署

由于我的 `Github Pages` 依赖于 `jekyll`，每次有较复杂的排版时总是会出现莫名其妙的问题，所以需要配置一个本地的预览环境。但是不想主机上安装过多的定制化功能，技术路线选择了 `Docker` 容器。



## 1. Jekyll

官网：[https://jekyllrb.com/](https://jekyllrb.com/)

汉化：[https://jekyllcn.com/](https://jekyllcn.com/)

`Jekyll` 是一个简单的博客形态的静态站点生产机器。它有一个模版目录，其中包含原始文本格式的文档，通过一个转换器（如 Markdown）和我们的 Liquid 渲染器转化成一个完整的可发布的静态网站，你可以发布在任何你喜爱的服务器上。`Jekyll` 也可以运行在 `GitHub Page` 上，也就是说，你可以使用 `GitHub` 的服务来搭建你的项目页面、博客或者网站，而且是完全免费的。

### 1.1. 目录结构

| 文件                                | 描述                                                                                                               |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| `_config.yml`                       | 保存配置数据。                                                                                                     |
| `_drafts`                           | drafts（草稿）是未发布的文章。                                                                                     |
| `_includes`                         | 你可以加载这些包含部分到你的布局或者文章中以方便重用。                                                             |
| `_layouts`                          | layouts（布局）是包裹在文章外部的模板。                                                                            |
| `_posts`                            | 这里放的就是你的文章了。                                                                                           |
| `_data`                             | 格式化好的网站数据应放在这里。                                                                                     |
| `_site`                             | 一旦 `Jekyll` 完成转换，就会将生成的页面放在这里（默认）。                                                         |
| `.jekyll-metadata`                  | 该文件帮助 `Jekyll` 跟踪哪些文件从上次建立站点开始到现在没有被修改，哪些文件需要在下一次站点建立时重新生成。       |
| `HTML`, `Markdown`, `Textile files` | 如果这些文件中包含 `YAML` 头信息部分，`Jekyll` 就会自动将它们进行转换。                                            |
| `Other Files/Folders`               | 其他一些未被提及的目录和文件如 `css` 还有 `images` 文件夹， `favicon.ico` 等文件都将被完全拷贝到生成的 `site` 中。 |



## 2. Docker

官网: [https://www.docker.com/](https://www.docker.com/)

`Docker` 容器与虚拟机类似，但二者在原理上不同。容器是将操作系统层虚拟化，虚拟机则是虚拟化硬件，因此容器更具有便携性、更能高效地利用服务器。容器更多的用于表示软件的一个标准化单元。由于容器的标准化，因此它可以无视基础设施（Infrastructure）的差异，部署到任何一个地方。



## 3. 部署

### 3.1. 下载 Docker

直接到官网下载，`Windows` 直接下载桌面版安装即可，会重启电脑并自动将命令添加在环境变量中。

### 3.2. 下载 Jekyll

```dockerfile
docker pull jekyll/jekyll
```

### 3.3. 使用 Jekyll

#### 3.3.1. 下载 Jekyll 博客模板

在 `GitHub` 上下载一个模板即可。

```dockerfile
git clone git@github.com:august295/august295.github.io.git
```

#### 3.3.2. 本地部署

```dockerfile
docker run -it --name my-jekyll -p 127.0.0.1:4000:4000 -v D:\\Github\\august295.github.io:/srv/jekyll jekyll/jekyll jekyll serve
```

- `my-jekyll`: 创建容器名称
- `4000:4000`: 容器的端口映射
- `D:\\Github\\august295.github.io`: `Windows` 本地路径
- `/srv/jekyll`: 映射路径
- `jekyll/jekyll`: 指定上述下载的 `Jekyll`
- `jekyll serve`: `Jekyll` 启动服务

![](/media/image/2022-02-20-Jekyll本地Docker部署/docker-jekyll-server.png)

看到服务网址即可在本机上访问：[http://127.0.0.1:4000/](http://127.0.0.1:4000/)

#### 3.3.3. 其他命令

```dockerfile
# 开启容器
docker start my-jekyll
# 重启容器
docker restart my-jekyll
# 关闭容器
docker stop my-jekyll
```

