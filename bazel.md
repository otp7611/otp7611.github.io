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

## cc_configure.bzl

rules_cc/cc/private/toolchain/cc_configure.bzl

调用cc_configure()后生成cc_autoconf_toolchains(name = "local_config_cc_toolchains")，cc_autoconf(name = "local_config_cc")

cc_autoconf_impl()自动配置编译器。

get_cpu_value()检测系统k8是linux, x64_windows是windows, 这里msys也是用cpu为x64-windows

load(":windows_cc_configure.bzl", "configure_windows_toolchain")配置windows编译器信息

使用"@rules_cc//cc/private/toolchain:BUILD.windows.tpl"这个模块来配置local_config_cc。

## repository_rule

environ,repository_ctx.getenv如果环境变量变化，这个依赖的仓库会被更新。

Specifies additional environment variables to be available only for repository rules。这个环境变量是通过bazel命令参数

```
--repo_env=
```

来传递的，且只对repository_rule规则有效。

--action_env=BAZEL_SH=/bin/bash  action_env配置的变量在action中有效，比如genrule，sh_binary











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

# 参数bazel命令参数输入配置

命令参数转化为flag, 各种flag组合成config_setting, 最后由select去选中满足所有flag的config_setting.

# rule()

rule的实现函数返回的是provider列表，provider的类型是不可重复的。

# toolchain

https://bazel.google.cn/versions/8.4.0/extending/toolchains

toolchain的选择是由platform constraints控制的。它不是直到从命令行参数传递的选择。每个rule()都可以通过参数toolchains指定多个toolchain. 每个toolchain可以通过select去指向不同具体的配置。

最后通过register_toolchains或--extra_toolchains把定义的toolchain()注册到bazel中。--platforms=//my_pkg:my_target_platform去选择platform constraints。

# bazel使用的工作目录与依赖库

bazel使用的工作目录可以通过--output_user_root来指定。如果项目中使用的依赖库是一样的（如果是链接文件生成的依赖库，则链接文件指向的目录也是一样），那么，确实可以通过复制“工作目录”来把数据迁移到另一台电脑上。但是，如果依赖库不一样，最常见的是就是链接文件指向的目录不一样。这个就会导致bazel把这个工程生成的hash值不一样，导致迁移到另一台电脑上出现需要重新下载的情况。

如果重新下载很慢，可以通过复制现有的下载的文件。举个例子，如果下载skia依赖库spirv_cross很慢，就需要复制

```
<工作目录>/<正常编译项目录的HASH值>/external/skia++dep_extension+spirv_cross
<工作目录>/<正常编译项目录的HASH值>/external/@skia++dep_extension+spirv_cross.marker
```

把这两个复制到

```
<工作目录>/<新编码项目录的HASH值>/external/
```

。

# 在--compilation_mode=fastbuild时会对生成的制品移除调试信息的分析

在bazel源码中./src/main/java/com/google/devtools/build/lib/rules/cpp/CppConfiguration.java:299

```
    this.stripBinaries =
        cppOptions.stripBinaries == StripMode.ALWAYS
            || (cppOptions.stripBinaries == StripMode.SOMETIMES
                && compilationMode == CompilationMode.FASTBUILD);
    this.compilationMode = compilationMode;
```

378行

```
  @Override
  public boolean shouldStripBinariesForStarlark(StarlarkThread thread) throws EvalException {
    CcModule.checkPrivateStarlarkificationAllowlist(thread);
    return stripBinaries;
  }

```

./src/main/java/com/google/devtools/build/lib/starlarkbuildapi/cpp/CppConfigurationApi.java:168

```
  @StarlarkMethod(name = "should_strip_binaries", useStarlarkThread = true, documented = false)
  boolean shouldStripBinariesForStarlark(StarlarkThread thread) throws EvalException;
```

./src/main/starlark/builtins_bzl/common/cc/link/link_build_variables.bzl:230

```
    if not must_keep_debug and cpp_config.should_strip_binaries():
        vars[LINK_BUILD_VARIABLES.STRIP_DEBUG_SYMBOLS] = ""
```

63行

```
    # Presence of this variable indicates that the debug symbols should be stripped.
    STRIP_DEBUG_SYMBOLS = "strip_debug_symbols",

```

这样就通过compilation_mode控制了strip_debug_symbols是否有定义.

在rules_cc使用这个定义

./external/rules_cc+/cc/private/toolchain/unix_cc_toolchain_config.bzl:1090

```
    strip_debug_symbols_feature = feature(
        name = "strip_debug_symbols",
        flag_sets = [
            flag_set(
                actions = all_link_actions + lto_index_actions,
                flag_groups = [
                    flag_group(
                        flags = ["-Wl,-S"],
                        expand_if_available = "strip_debug_symbols",
                    ),
                ],
            ),
        ],
    )
```

ld参数-S会去掉制品中的调试信息.

