源码编辑工具sublime_text

# 同时打开多个标签

单击默认情况下是以预览方式打开文件，此在标签上显示的文件名是斜体的。此时如果你点击其它标签，则会在同一个标签是关闭先前旧的文件，再去打开新文件。所以你要双击文件或双击标签。双击后，标签上的文件名是正的，此时你去打开新的文件，就是在新的标签上打开了。

修改配置改变这个默认行为,此时单击就不会打开文件了。

```
{
    "preview_on_click": false
}
```

# 基本sublime_text配置

https://www.sublimetext.com/docs/projects.html

https://www.sublimetext.com/docs/file_patterns.html

```
{
	"vintage_start_in_command_mode": true,
	"ignored_packages": [""],
	"tab_size": 4,
	"translate_tabs_to_spaces": true,
 	"detect_indentation": false,
	"update_check": false,
	"word_wrap": "false",
	"binary_file_patterns": ["*.node", "*.jpg",],
	"folder_exclude_patterns": [".git/", "//node_modules", "/home/data/mediaproj/testelectron/build"],
	"font_size": 12,
	"preview_on_click": true,
}
```

```
[
	{ "keys": ["ctrl+shift+s"], "command": "save_all" },	
	{ "keys": ["ctrl+tab"], "command": "switch_file", "args": {"extensions": ["cpp", "cxx", "cc", "c", "hpp", "hxx", "hh", "h", "ipp", "inl", "m", "mm"]} },
]

```



## 路径配置File Patterns

相对是在路径中搜索模式

以/开始是系统绝对路径

以//开始是相对于the project root项目根目录

# 列选择

```
    { "keys": ["alt+left"], "command": "move", "args": {"by": "subwords", "forward": false} },
    { "keys": ["alt+right"], "command": "move", "args": {"by": "subword_ends", "forward": true} },
    { "keys": ["alt+shift+left"], "command": "move", "args": {"by": "subwords", "forward": false, "extend": true} },
    { "keys": ["alt+shift+right"], "command": "move", "args": {"by": "subword_ends", "forward": true, "extend": true} },
    { "keys": ["alt+shift+up"], "command": "select_lines", "args": {"forward": false} },
    { "keys": ["alt+shift+down"], "command": "select_lines", "args": {"forward": true} },
```

alt+shift+方向