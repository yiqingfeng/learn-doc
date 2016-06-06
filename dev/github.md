# git与github相结合使用
----

在某些情况下，使用本地git更加方便。当我们在不同的工作地点却想要同步工作的时候，这时就可以借助线上平台github。

## 将线上github项目拷贝到本地

```
git clone github_href
```

## 将本地修改提交到github上
1. `git add . // 将所有文件添加进工作区`
2. `git commit -m "message" // 对本次修改进行提交`
3. `git log --abbrev-commit // 查询提交日志-简化版`
4. `git tag -a v0.0.1 f097726 // 给指定提交添加一个标签`
5. `git tag // 查询所有标签`
6. `git remote add origin git@github.com:yiqingfeng/PTMS.git // 连接到相关的github源上`
7. `git push origin master --tag // 将提交结果及标签推送到github上`
