---
title: hexo theme quite
abbrlink: 48616
date: 2021-05-07 00:00:00
---

### download
```sh
git clone https://github.com/QiaoBug/hexo-theme-quiet.git
```

### _config.yml
```yaml
theme: Quiet
```

### page
```yaml
index_generator:
  path: ''
  per_page: 9
  order_by: -date
```

### highlight
```
# 我的配置
highlight:
  enable: true
  line_number: false
  auto_detect: true
  tab_replace: ''
  wrap: true
  hljs: true
prismjs:
  enable: false
  preprocess: true
  line_number: true
  tab_replace: ''
```

### tag
进入根目录下的source文件夹下创建tag文件夹新建index.md文件
```markdown
---
title: tags
date: 2020-09-19 16:19:22
layout: "tags"
author: 79bk.cn
---
```

### about
进入根目录下的source文件夹下创建about文件夹新建index.md文件
```markdown
---
title: 个人简介
date: 2020-11-03
aubot: Cange-Q
portrait: 'https://cdn.jsdelivr.net/gh/duogongneng/MyBlogImg/imgIMG_7327.jpeg'
describe: '一个阳光快乐的BOY,在正合适的年龄里希望遇见正好的你。'
type: "about"
layout: "about"
author: 79bk.cn
---
```