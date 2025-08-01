vue

# 建立工程

## 配置代理

```
npm config set registry http://mirrors.cloud.tencent.com/npm/
npm config get registry
```



## 生成package.json

```
npm init -y
```

## 安装依赖

这些依赖在package.json中的dependencies和devDependencies指定

```
npm install
```

## 运行脚本

脚本在package.json中的scripts中指定

```
npm run xxx
```

# 构建不同参数的制品

在package.json的scripts中添加不能配置

```
"scripts": {
    "build": "build-only build-site",
    "build-only": "vite build",
    "build-site": "vite build --config vite.config.site.ts"
  }
```

npm run build会执行build-only，build-site

npm run build-only 会执行vite build

npm run  build-site会执行vite build --config vite.config.site.ts
