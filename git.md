git

# 生成和使用补丁git patch

```shell
git format-patch -1 --suffix=.root
git am <xxx.patch>
```

# 全并新仓库到本地仓库

https://stackoverflow.com/questions/1425892/how-do-you-merge-two-git-repositories

```
git remote add <origin> <origin-url>
git fetch
git merge --allow-unrelated-histories origin/master
```

