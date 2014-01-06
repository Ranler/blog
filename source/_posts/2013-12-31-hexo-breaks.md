title: hexo的换行符问题
date: 2013-12-31 21:20:44
categories: hexo
---

hexo使用markdown作为文章格式模板，但其默认的换行是采用[GFM的换行策略](https://help.github.com/articles/github-flavored-markdown)，
每一行对应html中的一个段落，而不是从latex那边习惯的多行对于一个段落的策略。
遂改之，但对js不熟悉，小研究了一下，发现hexo默认使用[markd.js](https://github.com/chjj/marked)做为markdown的renderer，
markd.js中有对换行策略的选项，并且默认使用的就是latex那种策略。。。

好吧，改过来，在hexo的源码`hexo/lib/plugins/renderer/markdown.js`第11行：

```
  breaks: true,
```

改为：

```
  breaks: false,
```

重新生成html，解决。

