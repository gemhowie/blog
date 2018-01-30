## 说明

我的个人博客：<https://gemhowie.github.io/blog>

## 致谢

基于 [mzlogin.github.io](https://github.com/mzlogin/mzlogin.github.io) 修改，感谢作者

## 修改内容

修改所有页面引用url的方式，方便在以下两种展示方式之间进行切换

GitHub Pages提供的两种访问方式：

* repo名为 `username.github.io` 的 master 分支，通过 `https://username.github.io` 访问
* 或任意repo名下的 gh-pages 分支，通过 `https://username.github.io/<reponame>` 访问，可以设置多个博客

clone到本地，执行带 baseurl 参数的命令 `bundle exec jekyll serve --baseurl=""` 即可正确在本地预览
