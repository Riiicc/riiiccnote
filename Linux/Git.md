
## git最小配置
```-- bash
# 配置全局（global）用户信息
git config --global user.name 'yourname'
git config --global user.email 'youremail'

--global 全局
--local  某个仓库 生效
--system 对系统所有登陆用户有效

# 显示相关配置可以用 --list
git config --list --local
git config --list --global
git config --list --system

优先级 local > global  


```
## 重命名文件
```
git add 添加到暂存区
git rm  移除暂存
git mv 移动/重命名

```

#### 同步提交
```
git commit -am 'message'

```

## git log
```
git log --oneline  简洁模式

git log -n4 最近四次提交

git log -n4 --online

git log --all --graph  图形化查看

```

## 图形界面
```
gitk

```

### 比较差异
```
git diff HEAD HEAD-1
git diff HEAD HEAD^

git diff commit comit

```

### 修改 commit message（本地仓库操作）
``` 
# 修改最近一次message
git commit --amend

# 修改任意一次提交
commit串为需要修改的提交的 父commit串 
git rebase -i <commit 串>  

先根据弹出文本修改重置类型 再修改commit message

将多个commit 合并为一个 同上 抹掉的commit 使用 squash替换pick

```