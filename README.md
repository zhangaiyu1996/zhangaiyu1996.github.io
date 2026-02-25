# 张爱宇的个人博客

基于 [Jekyll](https://jekyllrb.com/) 和 [GitHub Pages](https://pages.github.com/) 搭建的个人博客。

## 在线访问

[https://zhangaiyu1996.github.io](https://zhangaiyu1996.github.io)

## 本地运行

```bash
# 安装依赖
bundle install

# 启动本地服务
bundle exec jekyll serve
```

访问 `http://localhost:4000` 查看效果。

## 发布文章

在 `_posts/` 目录下创建 Markdown 文件，命名格式：

```
YYYY-MM-DD-文章标题.markdown
```

文件头部添加 Front Matter：

```yaml
---
layout: post
title: "文章标题"
date: YYYY-MM-DD HH:MM:SS +0800
categories: 分类名
---
```

## 技术栈

- Jekyll 静态站点生成器
- Minima 主题（自定义样式）
- GitHub Pages 托管
- jekyll-feed 插件（RSS）
