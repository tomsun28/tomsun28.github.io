---
layout: post
title: Issues Translate Action  
date: 2021-02-03
tag: github action
---

## Issues Translate Action   

  之前在公众号上看到了个[使用github action白嫖mac电脑使用]的文章，突然想起了自己之前没事做的自动识别翻译非英文issues的github action。  
  有些大佬同学拥有知名的开源项目(显然我还没有)，可能有时候会收到来自泰国新加坡印度尼西亚的同学的母语祝福issues，当然我们是看不懂需要手动google翻译，如果这时候有个机器人能自动识别非英文的issues把他翻译出来那感觉还是不错的。  
  先上效果：  
![这里放图片](/images/posts/issue_action/issues_demo.png)   

  这个action的原理也比较简单，issue动作触发action，读取issue comment内容，对其判断是否是英文，若不是则翻译出来，再调用github api评论一条翻译英文issue，然后再考虑下边界和死循环等。  
  咋使用呢，到github action市场搜索Issues Translator， https://github.com/marketplace/actions/issues-translator 里面就有对应的使用方法。或者直接在项目的`.github/workflows/`下创建 `issues-translator.yml`，填入以下内容：  

````
name: 'issue-translator'
on: 
  issue_comment: 
    types: [created]
  issues: 
    types: [opened]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: tomsun28/issues-translate-action@v2.3
````

就OK了，接下来就是要等泰国新加坡印度尼西亚的朋友们来祝福你了哦！  






