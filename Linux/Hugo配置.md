#  Hugo 配置教程   

官方安装文档 https://gohugo.io/installation/   


我的hugo博客访问地址  https://riiicc31.github.io/


# 安装  
https://github.com/gohugoio/hugo/releases  下载 extended 版本（理解为高性能版本）的 windows zip压缩包   

直接打开压缩吧执行 hugo.exe安装    
通过命令 `hugo version` 查看hugo版本信息   


`hugo new site cuttontail-blog` 创建新博客   

**文件夹说明**    
- archetypes：存放用 hugo 命令新建的 Markdown 文件应用的 front matter 模版
- content：存放内容页面，比如「博客」、「读书笔记」等
- layouts：存放定义网站的样式，写在layouts文件下的样式会覆盖安装的主题中的 layouts文件同名的样式
- static：存放所有静态文件，如图片
- data：存放创建站点时 Hugo 使用的其他数据
- public：存放 Hugo 生成的静态网页
- themes：存放主题文件
- config.toml：网站配置文件



# 配置 Hugo 主题  

主题备选网站  https://themes.gohugo.io/   

主题直接下载后放在 themes中

一般安装的 Hugo 主题的文件结构中都会有 `exampleSite` 文件夹，也是你在选择主题时参考的网站 demo    
比如exampleSite下有 content ,  static  和  config.toml 3 个文件，就找到你自己的站点跟目录下这对应的三个文件。在把对应目录中的内容分别复制过去



# 本地调试 
hugo server  



# 发布内容

hugo  命令可以将你写的 Markdown 文件生成静态 HTML 网页，生成的 HTML 文件默认存放在 public 文件夹中










# 参考文档   
https://zhuanlan.zhihu.com/p/396669087   

https://cuttontail.blog/blog/create-a-wesite-using-github-pages-and-hugo/


