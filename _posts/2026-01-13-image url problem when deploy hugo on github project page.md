---
title: 🌳image url problem when deploy hugo on github project pages
layout: post
author: cyven
tags: url hugo
categories: CS CS::Tech
---


> 最关键的是,需要定位到问题的命门,说白了还是需要抓住主要矛盾.

## 问题背景

Github Pages其实可以部署三种, personal page,organization page, project page
最常见的 `${username}.github.io` 这种是 personal page,这个是只能有一个的,就是所谓的需要建立一个同用户名的仓库

其实你也可以建立其他仓库 foo, 然后通过 `${username}.github.io/foo` 来访问

其次,GitHub page是可以绑定域名的,当 personal page 绑定了一个域名后, 比如我自己就是 `www.cybertheye.com`
那么foo这个page,就可以通过 `www.cybertheye.com/foo` 来访问

那么我有一个braindump站点,使用 Hugo 来搭建部署的.

我的工作流是,在 Emacs中使用 org-mode 来编写内容,然后使用 org-hugo 导出内容到 hugo项目文件中,
如果有图片那么会根据设置要么导出到 `static/` 文件夹下面,要么是 `page bundle`

然后上传到 github

在 github workflow 中


```shell
{% raw %}
  - name: Build the site
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "https://www.cybertheye.com/braindump/" \
            --cacheDir "${{ runner.temp }}/hugo_cache"
{% endraw %}
```


我已经把`baseURL`配置成带有 `/braindump/`,此时,我可以正常访问到posts了,

但是posts中如果有图片,显示不出来.

我打开网页浏览器的 inspector, 最后生成的html页面是`<img src="/ox-hugo/xxx.png">`,
其实只要改成`<img src="/braindump/ox-hugo/xxx.png>`
就能显示图片。

## 分析的过程

因为我已经设置了 baseURL, 但是img的url没有加上 `braindump`,

我查看了带图片的 md 文件, 图片格式是

{% raw %}
`{{< figure src="/ox-hugo/xxxx.png" >}}`
{% endraw %}

这里我一开始一直没有注意到 `/ox-hugo`的这个 `/`

我一直怀疑是 hugo或github page哪里配置不对,也许workflow需要再加上什么命令行参数

查看了 hugo 的 staticDir设置,如果把这个设置成 `static/braindump/`,只是改变了保存图片的路径,跟网络解析没有关系

后来查看了 hugo 的renderHook, 看这个名字感觉有点关系,但是最后发现,这里图片使用的是 figure shortcode,
而不是`![]()`这样的格式. renderHook 对hugo的figure shortcode不起作用

我甚至去看了 org-hugo 导出 md文件的源码,但是想想,这一步应该不是问题的关键

## 问题的命门

[资源路径解析]({% post_url 2026-01-14-url parse problem %})

ox-hugo 导出的图片路径,是 `/`开头的,那就说明它是从根域名开始解析的,
一般情况下,根域名其实就是对应的根目录,

但恰好我这里用的是 project page!!!根域名对应的不是根目录,解析就有问题了
但是此时问题的祸根就埋下了. 那就是 URL解析问题

因为我这里是使用 hugo 的 figure shortcode, 那么hugo是怎么从figure生成html的img,
这就是我们的解决切入点!

## 解决方法

> [Figure shortcode](https://gohugo.io/shortcodes/figure/#article)



```go
{% raw %}
  {{- $u := urls.Parse (.Get "src") -}}
  {{- $src := $u.String -}}
  {{- if not $u.IsAbs -}}
    {{- with or (.Page.Resources.Get $u.Path) (resources.Get $u.Path) -}}
      {{- $src = .RelPermalink -}}
    {{- end -}}
  {{- end -}}

  <img src="{{ $src }}"
{% endraw %}
```


这是默认的hugo把figure生成 img 标签的模板.
可以看到这里的 src 直接就是 `{{$src}}` 即使看不懂前面的语法,我们可以猜测,这就是
{% raw %}
`{{< figure src="/ox-hugo/xxxx.png" >}}`
{% endraw %}
的src部分

我们的目标是要 src变成 `/braindump/ox-hugo/xxx.png`这样的形式

1. 使用 hugo的模板语法,把`/braindump`加到 `$src`前面去
   这个可以使用 [urls.JoinPath方法](https://gohugo.io/functions/urls/joinpath/)

2. 先去除原来 `$src`的前缀 `/`,然后 img标签中 {% raw %}`src="{{ $src | relURL }}"`{% endraw %},
   relURL会生成相对`baseURL`的相对路径也就是 `/braindump/ox-hugo/xxx.png`了
   参考[relURL](https://gohugo.io/functions/urls/relurl/)

3. 在 hugo.toml 中设置 `"canonifyURLs=true"`,但是这种方式不再建议,后续版本可能会移除
   参考[Canonical URLs](https://gohugo.io/content-management/urls/#canonical-urls)
