编译和链接

# 有用的编译期检查选项

```
-Werror=exceptions
-Werror=unused-result
```



# 预编译头文件

参考https://clang.llvm.org/docs/PCHInternals.html

预编译的主要为了加快编译速度。如果你有个很大的头文件，对这个头文件进行预编译，会提高你的编译速度。这个技术在大型软件中用的比较多，比如QT。这里主要介绍clang的预编译头文件的方法。

预编译头对源代码是无入侵的（无需改动源代码），只要增加编译参数-include-pch即可。如果你是用gcc的话，这个参数都可以不用加，如果发现头文件相同位置目录下有个对应的.pch,则会自动使用此文件。

## 在工程使用预编译头文件

### 生成预编译头文件test.h.pch

```c++
clang++ -x c++-header -o headers/test.h.pch -c headers/test.h
```

参数-x c++-header表示输入是一个c++头文件。如果没有这个参数则会依据后缀来判定.h表示是c头文件，.hpp表示是c++头文件。

注意，预编译产生的中间文件是区分c和c++的，不能用c产生的预编译头文件用在编译.cpp源文件上。也不能用c++语法产生的预编译头文件用在编译c源文件上。

参数-emit-pch可选，如果输入为头文件，而会自动加入此参数表示是预编译头文件。

### 使用预编译头文件

```
clang -H -include-pch headers/test.h.pch -c test.cpp
```

参数-H可选，调试用。作用是用于输出所以引用的头文件.h，注意预编译头文件不算。举例：

```c++
#include "testpch.h"

int func() {
    return x;
}
```

如果你使用的预编译头文件test.h.pch，那么你不会看到输出test.h。如果你没有使用，则会有输出test.h。

参数-include-pch headers/test.h.pch表示使用预编译头文件。

### 在工具链中使用预编译头文件

预编译头文件保存了原头文件的路径和时间戳.clang在编译源文件时，如果发现预编译中间文件和原始头文件不匹配（原文件找不到或发生了修改），那么会提示pch出错。clang的这种机制能保证编译的正确性但同时也引入了复杂性。

由于预编译头文件存在这个限制，所以要求工具链：1，即时更新预编译头文件，2，不要在生成预编译中间文件后，就立即删除对应的头文件。要求2主要会发生在使用沙箱编译的情况，比如bazel默认就是用沙箱编译。

## 避免导致的mtime发生修改导致pch文件失效的方法

https://clang.llvm.org/docs/ClangCommandLineReference.html#cmdoption-clang-fpch-validate-input-files-content

```shell
-fpch-validate-input-files-content
```

当mtime发生改变时，比较文件内容。

这个行为和bazel是一样的。（https://github.com/bazelbuild/bazel/issues/21044）

目的就是为了避免，未修改文件，只是touch更新了一下时候却导致重新编译的问题。

