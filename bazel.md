bazel技术点

# 概述

bazel作为服务构建工具，它是一种描述型工具。所有的描述都在配置文件中，它不能在bazel命令行中传递其它构建参数。bazel命令行中只能打开配置文件中的开关。这个开关对应的内容还是在配置文件中的。可以理解为，所有都是服务构建的配置都是静态的，通过WORKSPACE.bazel,BUILD.bazel,*.bzl文件就可以确定所有分支，不依据于命令行中的参数。

# WORKSPACE.bazel与统一方式写label

WORKSPACE.bazel中的workspace(name = "sct")这一句让你能以统一方式写label.

如果引用的都是本工程内的可以直接以//来作引用。但是如果本工程作为其它工程的外部工程时，以//就是错的了，因为本工程的指的工程已经变化了。

# 引入第三方库

portaudio/instdir/BUILD.bazel

```shell
cc_import(
    name = "libportaudio-static",
    static_library = "lib/libportaudio.a",
)

cc_library(
    name = "portaudio",
    hdrs = glob(["include/*.h"]),
    includes = ["include"],
    linkstatic = True,
    visibility = [
        "//visibility:public",
    ],
    deps = [
        "//:libportaudio-static",
    ],
)

```

sct_mini/WORKSPACE

```shell
http_archive(
    name = "portaudio",
    sha256 = "f6b9e683fd408ed360e924fde007883cbda18ac737a3663c6082bbdb02ad3e40",
    strip_prefix = "instdir",
    urls = ["http://127.0.0.1:88/upl/portaudio.tar.bz2"],
)

```

sct_mini/BUILD

```
"@portaudio//:portaudio",
```



# 处理提示有循环依赖cycles detected



## 目录是工程的情况

```
ERROR: Failed to load .bzl file '//sct_mini:dep.bzl': possible dependency cycle detected.
ERROR: Error computing the main repository mapping: cycles detected during computation of main repo mapping

```

是因为sct_mini目录里是一个bazel工程含有(WORKSPACE), 但是使用了这个没有引入的工程，比如通过local_repository．

## 没有引入相应工程名的情况

```
ERROR: Failed to load Starlark extension '@sct//:dep.bzl'.
Cycle in the workspace file detected. This indicates that a repository is used prior to being defined.
The following chain of repository dependencies lead to the missing definition.
 - @sct
This could either mean you have to add the '@sct' repository with a statement like `http_archive` in your WORKSPACE file (note that transitive dependencies are not added automatically), or move an existing definition earlier in your WORKSPACE file.
ERROR: Error computing the main repository mapping: cycles detected during computation of main repo mapping

```

## 解决办法

即然是工程，就按工程的方式引用．

```shell
local_repository(
    name = "sct",
    path = "sct_mini",
)
load("@sct//:dep.bzl", "sct_dep")
sct_dep("sct_mini/")

```

# rules_cc

bazel使用rules_cc来配置C|C++编译环境。

可能通过环境变量来传参数。比如：--repo_env=CC=/llvm-project/instdir/bin/clang --repo_env=BAZEL_LINKLIBS=""

参考https://github.com/otp7611/rules_cc-example

支持的配置参数可以通过rules_cc/cc/private/toolchain/unix_cc_configure.bzl来确认。

# cc_toolchain_config

对编译工具链的定义是通过cc_common.create_cc_toolchain_config_info

cxx_builtin_include_directories不一定会进行include, 还需要手动通过flag进行添加features。

参考tensorflow的local_config_cuda/crosstool/cc_toolchain_config.bzl

# 备份项目与离线部署

项目备份直接tar打包。

下载的依赖库是在~/.cache/bazel/_bazel_chenc/cache/repos/v1/

这个路径可以通过repository_cache参数指定。

```
--repository_cache=<a path> default: see description
Specifies the cache location of the downloaded values obtained during the fetching of external repositories. An empty string as argument requests the cache to be disabled, otherwise the default of '<--output_user_root>/cache/repos/v1' is used

--output_user_root=<path> default: see description
The user-specific directory beneath which all build outputs are written; by default, this is a function of $USER, but by specifying a constant, build outputs can be shared between collaborating users.

```

output_user_root这个可以让每个项目使用不同的输出目录，互不干扰。
