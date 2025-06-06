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

