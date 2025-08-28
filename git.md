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

# 处理git push/pull超时问题

git一般是使用ssh地址，而ssh地址是使用端口号22的。如果希望访问github使用443端口，则需要指定端口号。可能通过命令参数-p 443或在文件~/.ssh/config进行配置。

https://docs.github.com/en/authentication/troubleshooting-ssh/using-ssh-over-the-https-port

Using SSH over the HTTPS port

```
$ ssh -T -p 443 git@ssh.github.com
Hi otp7611! You've successfully authenticated, but GitHub does not provide shell access.
```

~/.ssh/config

```
Host github.com
    Hostname ssh.github.com
    Port 443
    User git
```

```
$ ssh -T git@github.com
Hi otp7611! You've successfully authenticated, but GitHub does not provide shell access.
```

