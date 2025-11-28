msys2

# 路径

如果在msys2中使用非msys中的命令，如windows中的程序。这个程序使用的是windows中的路径，而不是msys中的路径。

msys会自动对路径进行转换，在parse的时候，把/e/mediaproj变成e:\mediaproj,会把多个路径中的冒号（:）变成windows中的逗号（,）

如果你不想它转换就写成//e/mediaproj或着设置环境变量

```
MSYS_NO_PATHCONV=1
MSYS2_ARG_CONV_EXCL="*"
```

```
MSYS2_ARG_CONV_EXCL="/LIBPATH;/home" "e:\program\clang+llvm-18.1.8-x86_64-pc-windows-msvc\bin\clang-cl.exe" -Wl,/LIBPATH:/e
```

## 举例：

在cmd中输入

```
Bat_To_Exe_Converter_4.2.exe /bat m.bat /exe m.exe /icon a/b/app.ico /invisible /x32
```

在msys就应该是

```
Bat_To_Exe_Converter_4.2.exe //bat m.bat //exe m.exe //icon a/b/app.ico //invisible //x32
```

msys bash会对所有以/开始的参数进行转换，对//就会变成/不作转换。
