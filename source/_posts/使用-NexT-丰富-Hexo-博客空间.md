---
title: 使用 NexT 丰富 Hexo 博客空间
date: 2018-07-06 15:25:44
tags:
	- Hexo
	- NexT
---

## 使用 NexT

进入 [NexT 官网](https://theme-next.iissnan.com/getting-started.html)，使用 **git 克隆**或者**直接下载 zip 文件**。

``` 
$ cd your-hexo-site
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```

在 `_config.yml` 中**启用 NexT 主题**。

``` yaml
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
# theme: landscape
theme: next
```

<!-- more -->

## 访客统计

1. 注册 LeanCloud 账号（不必翻墙），验证邮箱后 `新建应用`，等待 2 分钟后，进入 `存储` -> `创建 Class`，新建命名为 `Counter` 的 Class。这是为了与 NexT 主题对应，如无必要，不要改名。
2.  进入 `设置` -> `应用 key`，将 app id 和 app key 复制到 \next 主题目录下的 _config.yml 文件下对应位置。

``` yaml
# Show number of visitors to each article.
# You can visit https://leancloud.cn get AppID and AppKey.
leancloud_visitors:
  enable: true
  app_id: # <app id>
  app_key: # <app key>
```

3. 重新部署 Hexo，即可生效

## 字数统计

1. 安装插件

   ```
   npm install hexo-wordcount --save
   ```

2. 在 \next 主题目录下的 _config.yml 文件中，启用 wordcount

   ``` yaml
   # Post wordcount display settings
   # Dependencies: https://github.com/willin/hexo-wordcount
   post_wordcount:
     item_text: true
     wordcount: true
     min2read: true
     totalcount: true
     separated_meta: true
   ```

3. 重新部署 Hexo，即可生效。