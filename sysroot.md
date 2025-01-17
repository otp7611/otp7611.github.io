交叉编译与构建用于编译的sysroot

# 概述

x86_64平台也是可能需要交叉编译的，比如在ubuntu22.04编译出在ubuntu16.04上运行的程序。

# 生成git patch

```shell
git format-patch -1 --suffix=.root
```

# 对目标根文件目录打包

```shell
tar -cf /data/sysroot1604.tar /lib/ /usr/lib/ /usr/include

apt install libbz2-dev libgl-dev libglu1-mesa-dev libfontconfig-dev libwebp-dev libglvnd-dev libgles2-mesa-dev  libicu-dev  libfdk-aac-dev libfreetype6-dev

```

在容器内部，把需要的都打包进去

# 解压到构建用的sysroot目录

```shell
sysroots/x64-linux$ tar -xf /media/workspace/builder_data/sysroot1604.tar
```

# 做细节调整

```
-lc, -lrt, -ld一定是找libc.so, librt.so, libdl.so而不是其它文件，如果它是一个链接脚本，那么这个脚本就是相对于参数--sysroot=的值。
ln -sf /usr/lib/gcc/x86_64-linux-gnu/12/libgcc_s.so ./usr/lib/gcc/x86_64-linux-gnu/5/libgcc_s.so
ln -sf $PWD/lib/x86_64-linux-gnu/librt.so.1 ./usr/lib/x86_64-linux-gnu/librt.so
ln -sf $PWD/lib/x86_64-linux-gnu/libdl.so.2  ./usr/lib/x86_64-linux-gnu/libdl.so

```

# 错误error adding symbols: DSO missing from command line

如果出现了，表示你要显示连接这个文件。同一行已经提示要添加哪个文件。/usr/bin/ld: sdk/sysroots/x64-linux/lib/x86_64-linux-gnu/libpthread.so.0: error adding symbols: DSO missing from command line

# 编译应用程序

https://clang.llvm.org/docs/ClangCommandLineReference.html

```shell
-iwithsysroot<directory>
Add directory to SYSTEM include search path, absolute paths are relative to -isysroot
比如在cmake文件中通过：
add_definitions(-iwithsysroot /usr/include/freetype2)

```

# 查看导出的符号

```
readelf --wide  -s exec_app | grep GLIBC_2.34
```

# golang交叉编译

```
CC=clang go build -v -x -o app *.go
```

