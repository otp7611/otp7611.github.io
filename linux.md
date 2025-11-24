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
unset https_proxy HTTPS_PROXY HTTP_PROXY http_proxy
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

```
* * * * * root top -b -n 1 >> /tmp/log.test.$(date +\%Y\%m\%d) 2>&1 
```

crontab 中 `%` 是特殊字符，需要用 `\` 转义（写成 `\%`），否则会被解析为换行符。

若任务一天内执行多次（如每小时），但仍想每天合并到一个日志文件，上述命令同样适用（同一天的 date 输出相同，会追加到同一个文件）。

注意添加到/etc/cron.d/文件权限644,和所有者。

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
 SHELLPID=`echo $$`
 sudo sh -c "echo $SHELLPID >/sys/fs/cgroup/learnconfig/cgroup.procs"
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
grep -rlIi 'AAAA' . | xargs sed -i 's/AAAA/BBBB/g'
grep ts c.m3u8  | sed  's/^/file /g' > c.filelist
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

vim按百分比跳转。按5，0，％，就会跳到50％的位置。

# core

参考https://www.baeldung.com/linux/apport-enable-disable

linux程序崩溃就会被系统内核捕捉，并读取/proc/sys/kernel/core_pattern中配置，如果是配置

```
core
```

则会在进程工作目录下生成

```
core.<pid>
```

如果是

```
|/usr/share/apport/apport -p%p -s%s -c%c -d%d -P%P -u%u -g%g -- %E
```

则系统会把core数据流发给/usr/share/apport/apport这个程序处理。

这个过程可以通过/etc/init.d/apport来确认。

/usr/share/apport/apport这个文件在ubuntu中的软件包: apport (2.20.11-0ubuntu82.10) https://packages.ubuntu.com/jammy/apport

如果是使用了apport那么应用生成的core在

/var/crash 如果是deb包中程序

/var/lib/apport/coredump 如果不是.

日志在

```
/var/log/apport.log
```

## 查看apport服务状态

```
sudo systemctl status apport.service
```

## 禁用apport

/etc/default/apport

```
# set this to 0 to disable apport, or to 1 to enable it
# you can temporarily override this with
# sudo service apport start force_start=1
enabled=0
```

# bash脚本

## 脚本中的|| true

In Bash, the *|| true* construct is used to ensure that a command does not cause the script to exit when it fails. This is particularly useful when the *set -e* option is enabled, which makes the script exit immediately if any command returns a non-zero exit status.

```
#!/bin/bash

set -e # Exit on any command failure

# This command fails, but the script continues because of '|| true'
ls nonexistent_file || true

echo "Script continues despite the previous command failing."
```



# 查看系统日志

```
查看最后的启动时间who -b
查看最后的启动时间last reboot | head -1
查看日志
journalctl --since "2025-10-30 00:00:00"


```

确认出错设备

```
 lspci -nn | grep 00:02
00:02.0 PCI bridge [0604]: Intel Corporation Xeon E7 v4/Xeon E5 v4/Xeon E3 v4/Xeon D PCI Express Root Port 2 [8086:6f04] (rev 01)

```



```
-- Boot b7b80de5a2fa4c26b772b793688591cf --
表示系统启动开始。
```

pci故障

```
Oct 30 13:20:25 pc kernel: pcieport 0000:00:02.0: AER: device recovery failed
Oct 30 13:20:25 pc kernel: pcieport 0000:00:02.0: AER: Multiple Uncorrected (Fatal) error message received from 0000:00:02.0
Oct 30 13:20:25 pc kernel: pcieport 0000:00:02.0: PCIe Bus Error: severity=Uncorrected (Fatal), type=Transaction Layer, (Requester ID)
Oct 30 13:20:25 pc kernel: pcieport 0000:00:02.0:   device [8086:6f04] error status/mask=00004020/00000000
Oct 30 13:20:25 pc kernel: pcieport 0000:00:02.0:    [ 5] SDES                  
Oct 30 13:20:25 pc kernel: pcieport 0000:00:02.0:    [14] CmpltTO                (First)
Oct 30 13:20:25 pc kernel: nvidia 0000:03:00.0: AER: can't recover (no error_detected callback)
Oct 30 13:20:25 pc kernel: snd_hda_intel 0000:03:00.1: AER: can't recover (no error_detected callback)

```

GPU故障

```
Oct 30 09:49:36 pc kernel: NVRM: GPU at PCI:0000:03:00: GPU-44ec53bd-277c-984f-fa9f-cded30fbfd5d
Oct 30 09:49:36 pc kernel: NVRM: Xid (PCI:0000:03:00): 79, pid='<unknown>', name=<unknown>, GPU has fallen off the bus.
Oct 30 09:49:36 pc kernel: NVRM: GPU 0000:03:00.0: GPU has fallen off the bus.
Oct 30 09:49:36 pc kernel: NVRM: A GPU crash dump has been created. If possible, please run
                           NVRM: nvidia-bug-report.sh as root to collect this data before
                           NVRM: the NVIDIA kernel module is unloaded.

```

# 使用163邮箱为中继发邮件

## 安装

```
sudo apt install mailutils bsd-mailx
sudo cp /etc/postfix/main.cf /etc/postfix/main.cf.bak
```

准备好配置后要运行service postfix start来开启postfix

## 修改postfix配置

把旧配置smtp_tls_security_level, relayhost注释掉，然后添加：

```
relayhost = [smtp.163.com]:465
smtp_sasl_auth_enable = yes
smtp_sasl_security_options = noanonymous
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_use_tls = yes
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
smtp_tls_wrappermode = yes
smtp_tls_security_level = encrypt
smtputf8_enable=no
```

## 增加验证信息

```
sudo vim /etc/postfix/sasl_passwd
[smtp.163.com]:465 otp7611@163.com:<后台拿到的验证码,不是登录邮箱用的>
```

```
sudo postmap /etc/postfix/sasl_passwd
生成/etc/postfix/sasl_passwd.db
postman只能读取/etc/postfix/sasl_passwd.db
```

```
sudo chmod 600 /etc/postfix/sasl_passwd*
```

## 使用mail命令时，自动添加发件邮箱

bsd-mailx中的mail命令会使用/etc/mail.rc配置.

```
/etc/mail.rc
set from=<发件邮箱>
```

mailutils中的mail只能通过命令参数传递

```
-a FROM:<源邮箱>
```

## 测试发送邮件

```
sudo sh -c 'echo >/var/log/mail.log' && echo '111' | mail -a FROM:<源邮箱> -s '163relay-ssl-en-file' -A /etc/postfix/main.cf <目的邮箱>
```

## SMTPUTF8问题

```
 status=bounced (SMTPUTF8 is required, but was not offered by host smtp.163.com
```

添加参数

```
smtputf8_enable=no
```

然后在命令行中就可以使用中文主题了。

# 容器支持中文输入

```
sudo docker run -dit --name testemail ubuntu:22.04
sudo docker exec -it testemail /bin/bash
```

```
输入locale
如果看到POSIX就说明没有配置为支持中文输入模式
输入locale -a
应该要看到en_US.utf8和zh_CN.utf8
```

## 安装中文环境

```
apt install language-pack-zh-hans fonts-wqy-microhei fonts-noto-cjk locales
locale-gen en_US.UTF-8 zh_CN.utf8
fc-cache -fv
```

如果没有中文字体，会显示为方块。

## 配置环境变量

```
update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
echo 'export LC_ALL=en_US.UTF-8' >> ~/.bashrc
echo 'export LANG=en_US.UTF-8' >> ~/.bashrc
echo 'export LANGUAGE=en_US.UTF-8' >> ~/.bashrc
```

如果没有在bashrc中配置LC_ALL,你是看不到中文，甚至连方块都没有。

## 验证

```
locale
locale -a
curl -k  https://www.baidu.com
最后输入中文试一下。
```

