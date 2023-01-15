# Linux 速查

## 文件权限 
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/Linux文件权限.jpg)


```bash

sudo sed -e 's|^mirrorlist=|#mirrorlist=|g' \
         -e 's|^#baseurl=http://mirrors.tuna.tsinghua.edu.cn|baseurl=https://101.6.15.130|g' \
         -i.bak \
         /etc/yum.repos.d/CentOS-*.repo
```






## 参考资料
> - [Linux Tools Quick Tutorial](https://linuxtools-rst.readthedocs.io/zh_CN/latest/index.html)
> - [命令行的艺术GitHub](https://github.com/jlevy/the-art-of-command-line/blob/master/README-zh.md)
