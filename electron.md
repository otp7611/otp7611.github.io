electron

# 安装

```
export ELECTRON_MIRROR=https://npmmirror.com/mirrors/electron/
```

一定要有这个，不然出现连接错误。

## windows

```
%USERPROFILE%\.npmrc 内容为
registry=https://registry.npmmirror.com/
```

处理错误

```
mismatch detected for 'RuntimeLibrary': value 'MT_StaticRelease' doesn't match value 'MD_DynamicRelease'
```

```
MT_StaticRelease:
This signifies that the Multi-threaded (/MT) runtime library is being used, and it is a static release build. In this configuration, the C/C++ runtime library is statically linked into the executable or library. This means that the necessary runtime code is directly embedded into the final binary, making it self-contained and not reliant on external runtime DLLs.
MD_DynamicRelease:
This signifies that the Multi-threaded DLL (/MD) runtime library is being used, and it is a dynamic release build. In this configuration, the C/C++ runtime library is dynamically linked, meaning the application or library will depend on external C/C++ runtime DLLs (e.g., msvcr*.dll) that must be present on the system where the application runs.
```

node的插件编译系统node-gyp使用是/MT, 所以在bazel使用clang-cl时，也要/MT。

node的插件编译系统默认使用Release模式，所以在bazel使用clang-cl时，也要Release模式。

### node-gpy使用clang-cl

```
      "msbuild_toolset": "ClangCL",
      "msvs_settings": {
        "VCCLCompilerTool": {
          "AdditionalOptions": ["/std:c++20"],
          "ExceptionHandling": 1,
        }
      }
```

### 环境变量

PYTHON这个变量一定在高级系统设置中设置，值为E:\Python\python3.exe。不然node-gyp找不到。

BAZEL_LLVM bazel不要使用vs中的llvm，要去官网下载。不然只能编译出x64的obj文件，会与x86的obj文件冲突，不能打包成lib库。

```
export BAZEL_LLVM="E:\program\clang+llvm-18.1.8-x86_64-pc-windows-msvc"
```

# 调试

windows崩溃输出的dump在%APPDATA%\应用名\Crashpad

```
C:\Users\<username>\AppData\Roaming\vncdaemon\Crashpad
```

```
 gdb --args ./node_modules/electron/dist/electron .
```



