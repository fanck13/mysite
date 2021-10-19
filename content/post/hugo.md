---
title: "Hugo"
date: 2020-08-09T17:42:34+08:00
draft: false
---

* 添加文件
  ```sh
  hugo new [file name]
  ```
* 添加 theme
  ```sh
  cd theme
  git submodule add https://github.com/pacollins/hugo-future-imperfect-slim.git
  ```

* 启动服务
  `hugo server --theme=binario --buildDrafts --watch`
  `--watch`:修改自动更新

* 部署
  `hugo --theme=binario --baseUrl="https://fanck13.github.io/"`

