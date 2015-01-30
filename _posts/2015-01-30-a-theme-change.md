---
layout: post
title: Theme Change
category: others
keywords: Theme Github
---
这是一个 jekyll的主题,直接修改 yml文件中的设置即可使用

clone https://github.com/onlyfu/logs

修改为 theme 去掉了 统计和分享代码

##目标 
    作为个人总的Github代码和项目展示使用 
    去掉了seo优化和分享功能
    个人认为这个主题不适合做个人blog
    当然一切随意

##使用

<pre class="prettyprint linenums">
excerpt_separator: "<!-s-more-->"
#这个是输出摘要的标示符
#如果使用做个方法需要把 index.html中
{% raw %} {{ post.content | strip_html | truncate: 175 }} {% endraw %}
修改为{% raw %} {{ post.excerpt }}{% endraw %}
#即以前的都会解析成html后输出
</pre>


请删除 CNAME,ico 和修改yml文件 

文章随意处理 但保留请署名和保留链接 

##注意 
如果使用本主题 作为其他的用途请询问 [@onlyfu](https://github.com/onlyfu) 作者本人

##jekyll 特点

###简单
不需要数据库，不需要评论功能，不需要不断的更新版本——只用关心你的博客内容

###静态
Markdown (或 Textile), Liquid, 和 HTML & CSS 构建可发布的静态网站

###博客形态
支持自定义地址，博客分类，单页应用，博客为单位存储，以及自定义的布局设计

