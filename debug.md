软件调试

# gdb

## 设置print长度

```
printf "%s\n", string
set print elements nnn

```

## 生成core文件

### 在gdb内

```
(gdb) help generate-core-file Save a core file with the current state of the debugged process. Argument is optional filename. Default filename is 'core.<process_id>'.
(gdb) generate-core-file

```



### 使用命令

```
gcore <pid>
```

# cdb

```
"C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\cdb.exe" -lines -y "e:\pathtopdbfile;srv*http://msdl.microsoft.com/download/symbols"  -i app.exe -z e:\some.dmp
.ecxr ; kb ;
dt /r 树形显示成员变量
 dt 变量　成员变量a.成员变量a的成员变量
 dt this a.exeInfos_.
dv 局部变量
~*kb 所有线程堆栈
~46 s 设置为线程号46
.frame 4  切帧栈４
 . srcpath (设置源路径

```

