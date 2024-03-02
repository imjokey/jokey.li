+++
author = "Jokey Li"
title = "使用Github Pages实现\"服务CNAME\"功能"
date = "2020-09-22"
description = "利用免费的 Github pages 实现了一个简单的WEB页面跳转（站点服务是以域名区分情况下）功能。"
featured = true
tags = [
    "Github",
    "网页重定向"
]
categories = [
    "实用技巧",
]
series = ["网路技术"]
thumbnail = "images/materials/blackcat.jpeg"
+++

## "服务CNAME" 需求背景

> **[CNAME解析](https://zh.wikipedia.org/zh-cn/CNAME%25252525E8%25252525AE%25252525B0%25252525E5%25252525BD%2525252595)**：当一个 DNS 解析服务器在查询某域名遇到 CNAME 记录时，它会重启查询，查询并返回 CNAME 目的域名对应的 IP。

当有域名自动跳转（CNAME）到某一个指定 WEB 服务网站（单域名）的需求时，一般是在域名注册商那里添加 CNAME 解析就可以了， **但是如果目标站点与多个其他站点服务部署在同一台服务器上，且站点服务是以域名区分（根据不同域名区分不同服务，但公用同一个IP和端口）的情况时，由于 CNAME 解析主要的作用只是映射出 CNAME 的目标域名的 IP 地址，本身不会做域名路由跳转，这样的话就不能正常跳转到指定域名的网站页面了**，常见做法是给原域名搭建一个WEB网站服务，然后再通过这个WEB服务专门去做重定向跳转，但是仅仅为了一个页面跳转的功能就再搭建一个web服务的话，就有点太浪费成本了，有没有比较简便的方式呢？实际上我们可以利用免费的Github Pages服务来实现这个需求。

## 解决思路和方法步骤


> **[GitHub Pages](https://docs.github.com/en/pages)**: 是GitHub提供的一个网页寄存服务，可以用于存放静态网页，包括博客、项目文档或者书籍等。同时也提供免费的自定义域名功能。

我们可以利用Github Pages做一个静态页面，使用Github自定义项目域名的功能给项目绑定域名（原域名），再在项目的静态页面中使用代码跳转方式到新域名的站点，这样就完美的实现了“服务CNAME“的跳转需求，具体做法如下：

1.在Github上新建一个项目仓库，然后在仓库里添加一个简单的名为 index.html 的静态网页文件。

2.根据Github的自定义项目域名的说明文档配置访问解析，配置自定义域名详见官方文档 [Configuring a custom domain for your GitHub Pages site - GitHub Docs](https://help.github.com/en/github/working-with-github-pages/configuring-a-custom-domain-for-your-github-pages-site) .

3.编辑修改 index.html，可以使用 `HTML meta` 标签重定向和 `JavaScript` 脚本跳转两种方式（双保险），例如我想在访问 [`http://example.com（原域名）`](http://example.com)时重定向访问[`http://example2.com（跳转域名）`](http://example2.com)的服务，则可以在  index.html 中修改添加如下代码：

```html
<html xmlns="http://www.w3.org/1999/xhtml"> 
<head> 
<!--使用html meta标签重定向--> 
<meta http-equiv="refresh" content="0; url=http://example2.com"/> 
...
<script type="text/javascript">
 
    window.location.href = "http://example2.com" //使用js跳转
 
</script>
</head>
...
```

待配置的 Git 仓库自定义域名解析生效后，借用这个机制就可以实现免费的网站服务的重定向了。

## 总结

本文通过利用免费的 Github pages 实现了一个简单的 WEB 页面跳转（站点服务是以域名区分情况下）功能，原理虽简单，但省去了专门配置额外站点的成本，并且利用了 Github 托管的可用性和安全性优势，也不妨是一个便捷的解决方法。

## 参考材料

- [CNAME记录 - 维基百科，自由的百科全书](https://zh.wikipedia.org/zh-cn/CNAME%25252525E8%25252525AE%25252525B0%25252525E5%25252525BD%2525252595)
- [Configuring a custom domain for your GitHub Pages site - GitHub Docs](https://help.github.com/en/github/working-with-github-pages/configuring-a-custom-domain-for-your-github-pages-site)