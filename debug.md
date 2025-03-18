软件调试

# gdb

## 设置print长度

```
printf "%s\n", string
set print elements nnn

```

## 查看std::shared_ptr指向的对象

```
*packet.__ptr_
```

pakcet是一智能指针.

```
(gdb) p *(*packet.__ptr_).packet_.__ptr_
$6 = {buf = 0x74b7f0078700, pts = 1696427000, dts = 1696427000, data = 0x74b7e066a940 "", size = 384, stream_index = 0, flags = 0, 
  side_data = 0x0, side_data_elems = 0, duration = 0, pos = -1, opaque = 0x0, opaque_ref = 0x0, time_base = {num = 1, den = 1000000}}
```

## 查看指定内存地址数据

```
(gdb) x /10b 0x74b7e066a940
0x74b7e066a940: 0x00    0x00    0x00    0x01    0x61    0xe0    0x01    0x80
0x74b7e066a948: 0x03    0x3e
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

