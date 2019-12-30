---
title: About Blog
date: 2019-11-27 15:38:46
tags: hexo
categories: "其他"
---

这里会写一些关于我使用 next 遇到的问题及解决方案，也许我的不是解决方案，只是我能想到的妥协办法。

<!-- more -->

## 关于导航问题

导航点击之后进入到一个类似于 `Cannot GET /categories/` 的页面。需要手动创建：

```shell
hexo new page categories
```

这样就会在 source 目录下生成 categories 文件夹，之后修改 index.html 文件

```markdown
---
title: 分类
date: 2019-11-27 15:38:46
type: "categories"
---
```

## 文件名问题

文件名字不允许大写，域名链接全都是小写，如果文件名字大写则会对应不到目标文件，从而404。

## 访问人数问题

如果要打开访问人数，只需要在 _config.yml 中配置 busuanzi_count 的 enable: true 就可以同时看到访问人数和访问次数。

如果想只看到访问人数，或者访问次数，只需要配置 total_visitors 或者 total_views 就可以。

如果想要自定义，则可以将上述两个配置选项置为 false，并在 themes\next\layout\_partials\footer.swig 中添加如下代码。

```html
{%- if theme.busuanzi_count.enable %}
  <!-- 与前文分离的丨 -->
  <span class="post-meta-divider">|</span>

  <!-- 访问次数 -->
  <span id="busuanzi_container_site_pv">
      本站总访问量<span id="busuanzi_value_site_pv"></span>次
  </span>

  <!-- 访问人数 -->
  <span id="busuanzi_container_site_uv">
    本站访客数<span id="busuanzi_value_site_uv"></span>人次
  </span>
{%- endif %}
```

该算法由[不蒜子](http://ibruce.info/2015/04/04/busuanzi/)提供。

## 关于图片

新建文章时，在相同目录下创建同名文件夹

打开站点配置文件 _config.yml，搜索 post_asset_folder 字段，设置其值为 true

安装 hexo-asset-image：`npm install hexo-asset-image --save`

此时 `hexo new "fileName"` 会在 /source/_posts 目录下创建同名的文件夹

只需在 md 文件里使用 `![title](图片名.jpg)` ，无需路径名就可以插入图片。

## 关于多 tags

可以使用如下两种格式：

1. 使用中括号

    ```yaml
    tags: [tag1, tag2]
    ```

2. 使用 -

    ```yaml
    tags:
    - tag1
    - tag2
    ```
