linux

# 开机启动服务

把需要启动的命令放入/etc/rc.local

这个脚本在systemd中需要开始服务才能运行。对应的服务为：

```
sudo systemctl status rc-local
```

# cgroup2中使用cgroup1接口

如果系统使用的是cgroup2，那么想用cgroup1的接口就需要自己挂载上去。

```
mkdir /sys/fs/cgroup/cpuacct
mount -t cgroup -o cpuacct none /sys/fs/cgroup/cpuacct
echo "/var/core/core-%e-%p-%t" > /proc/sys/kernel/core_pattern
```

# 查看系统DNS

```
dig <域名>
nsloopup <域名>
```

# 查看分区使用情况

```
sudo du -d 1 -h / --exclude=/run --exclude=/dev/shm --exclude=/media/workspace --exclude=/home --exclude=/proc --exclude=/snap
```

# 配置apt源

```shell
sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
sed -i 's/security.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
```

# cron定时任务

文件名不能有后缀。

/etc/cron.d里面的文件可以不用可执行权限。

```
# m h dom mon dow user  command
0 16 * * * root find /data/targetfolder/ -type f -mtime +14 [加此参数表示删除  -delete]
```

其它如/etc/cron.daily是需要的。你可以通过run-parts --test来检测一个任务文件是否有效。比如：run-parts --test /etc/cron.daily定时任务文件，要是可执行文件且不能有后缀，（最容易犯的错误是有后缀.sh, 合法的名字是不能有后缀）

```
内容就是shell脚本
```

# 查看多文件里的日志信息

```
grep -arih aaaaa /logs/ | sort | less
```

-a 把处理二进制数据，否则遇到二进制数据grep 会停止

-r 递归

-i 忽略大小写

-h 不显示匹配文件名

-n  显示匹配号

# 查看进程的环境变量

```
cat /proc/5909/environ | while IFS= read -r -d '' v; do echo $v; done
```

```
cat /proc/217632/environ | xargs -0 echo
```

IFS是read中用于控制分词的，-d表示read读到null后，read结束这次读。

# 限制进程内存使用

配置cgroup

```
sudo mkdir /sys/fs/cgroup/learnconfig
sudo sh -c 'echo 8000000000 >/sys/fs/cgroup/learnconfig/memory.max'
cat /sys/fs/cgroup/learnconfig/memory.max
```

设置进程

 ```
 echo $$
 sudo sh -c 'echo 3924 >/sys/fs/cgroup/learnconfig/cgroup.procs'
 cat /proc/self/cgroup
 ```

# 分割保存文件

```
split -b 1G /media/workspace/github/llvm-project-18.1.8.src/llvm-project-18.1.8-d1447217.tar llvm-project-18.1.8-d1447217.tar
```

# 管理应用输出工具rotatelogs

```shell
nohup bash -c 'python3 main.py -1 2>&1 | rotatelogs -l /data/node_python.%Y-%m-%d-%H_%M_%S 30M' >/dev/null 2>&1 &
```

# 文本处理

```
grep -rlI 'AAAA' . | xargs sed -i 's/AAAA/BBBB/g'
```



# vim配置

```
~/.vimrc 
filetype plugin indent on
set tabstop=4
set shiftwidth=4
set expandtab
set mouse=
set ttymouse=

```







