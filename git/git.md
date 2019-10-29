## 查看所有的分支

```bash
git branch -a
```



## 创建本地分支

```bash
git checkout -b 新分支名
```



## 推送本地分支到远程仓库

```
git push --set-upstream origin 分支名
```



## 将远程git仓库里的指定分支拉取到本地（本地不存在的分支）

```
git checkout -b 本地分支名 origin/远程分支名
```

