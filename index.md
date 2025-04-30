音视频技术漫游

# 实用工具

markdown编辑工具Typora

图片编辑工具gimp

源码编辑工具sublime_text

## 文本处理

```
grep -rlI 'AAAA' . | xargs sed -i 's/AAAA/BBBB/g'
```



## 管理应用输出工具rotatelogs

```shell
nohup bash -c 'python3 main.py -1 2>&1 | rotatelogs -l /data/node_python.%Y-%m-%d-%H_%M_%S 30M' >/dev/null 2>&1 &
```



[json串解析工具jq](/jq)

[推流工具obs](/obs)

# 基础理论

[线性代数](/linearalgebra)

# 视频编码

[音视频实用工具ffmpeg](/ffmpeg)

[视频编码初探](/media)

# 网络传输

[代理服务器nginx](/nginx)

[rtmp和传输h265视频](/rtmpandh265)

# 大数据

[ES的使用](/es)

[grafana数据可视化](/grafana)

# 服务构建

[源码管理工具git](/git)

[bazel构建工具](/bazel)

[编译和链接](compileandlink.md)

[交叉编译与构建用于编译的sysroot](/sysroot)

[java工具链](/java)

[软件调试](/debug)

# 系统相关

[linux](/linux)
