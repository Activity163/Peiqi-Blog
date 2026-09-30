---
title: Hexo 写作效率小技巧
date: 2026-09-29 12:05:00
categories:
  - 教程
tags:
  - Hexo
  - 效率
---

## 一键新建文章

每次都敲 `npx hexo new "标题"` 太长了,可以在 `package.json` 里加个脚本:

```json
{
  "scripts": {
    "new": "hexo new",
    "dev": "hexo server",
    "build": "hexo generate"
  }
}
```

之后就能用 `npm run new "文章标题"`、`npm run dev` 这样的短命令了。

## 草稿功能

写一半的文章不想发布,可以放进草稿箱:

```bash
npx hexo new draft "未完成的文章"   # 存放在 source/_drafts/
npx hexo server --draft            # 本地预览时显示草稿
npx hexo publish "未完成的文章"     # 写完后发布
```

## Front-matter 常用字段

```yaml
---
title: 文章标题
date: 2026-09-29 12:00:00
categories:
  - 教程
tags:
  - Hexo
  - 测试
description: 用于 SEO 和首页摘要的描述文字
top_img: false      # 关闭顶部大图
aside: true         # 是否显示侧边栏
---
```

## 快速摘要

在正文里插入 `<!-- more -->`,它之前的内容会作为首页列表的摘要展示,不用再手写 description。

## 图片引用

把图片放进 `source/images/` 目录,在文中这样引用:

```markdown
![图片说明](/images/example.png)
```

掌握这些小技巧之后,写作流程会顺畅不少。
