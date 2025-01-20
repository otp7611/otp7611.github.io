java工具链

# dexdump 查看dex内容

```shell
dexdump classes.dex
```

# javap查看javac编译后的文件

```
javap -s -cp . cn.gosct.jni.SctJni
```

# javac编译java文件

```
javac -d . cn/gosct/jni/SctJni.java
```

# javah生成jni头文件

```
javah cn.gosct.jni.SctJni
```

