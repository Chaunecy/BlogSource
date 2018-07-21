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

## 评论系统

### 使用 Disqus

到 Disqus 官网 注册账号，取一个全网惟一的 shortname，后面的 `Website URL` 填写你的博客网址全称。

在 NexT 的配置文件中添加 shortname 即可。

``` yaml
# Disqus
disqus:
  enable: false
  shortname: # your shortname
  count: true
```



不足之处是必须要有 Facebook、Twitter 或者 Google Plus 账号才能评论。

### Valine

同样是在 LeanCloud 托管数据。新建应用，复制 App ID 和 App Key。

``` yaml
# Valine.
# You can get your appid and appkey from https://leancloud.cn
# more info please open https://valine.js.org
valine:
  enable: false
  appid: # your leancloud application appid
  appkey: # your leancloud application appkey
  notify: false # mail notifier , https://github.com/xCss/Valine/wiki
  verify: false # Verification code
  placeholder: # comment box placeholder
  avatar: mm # gravatar style
  guest_info: nick,mail,link # custom comment header
  pageSize: 10 # pagination size
```

好处：不用登录，快速评论。

坏处：评论质量可能不高。

### Gitment

在 [此处](https://github.com/settings/applications/new) 新建应用，生成 Client ID 和 Client sceret（之后要用）。

在 NexT 的配置文件中补全下述内容：

``` yaml
# Gitment
# Introduction: https://imsun.net/posts/gitment-introduction/
# You can get your Github ID from https://api.github.com/users/<Github username>
gitment:
  enable: true
  mint: true # RECOMMEND, A mint on Gitment, to support count, language and proxy_gateway
  count: true # Show comments count in post meta area
  lazy: false # Comments lazy loading with a button
  cleanly: true # Hide 'Powered by ...' on footer, and more
  language: # Force language, or auto switch by theme
  github_user: # MUST HAVE, Your Github ID
  github_repo: # MUST HAVE, The repo you use to store Gitment comments
  client_id: # MUST HAVE, Github client id for the Gitment
  client_secret: # EITHER this or proxy_gateway, Github access secret token for the Gitment
  proxy_gateway: # Address of api proxy, See: https://github.com/aimingoo/intersect
  redirect_protocol: # Protocol of redirect_uri with force_redirect_protocol when mint enabled
```

> 可能会出现 Error: validation failed 错误，这是因为文章 id 长度超过了 50 个字符。把 \themes\next\layout\_third-party\comments\gitment.swig 文件中的 id: window.location.pathname 改为 id: '{{ page.title }}'，即可奏效。

