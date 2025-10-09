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

# import路径配置

tsconfig.json中的paths配置，与vite.config.js中的resolve.alias配置是相互独立的。tsconfig.json是给tsc用的，vite.config.js是给vite打包器用的。路径配置在这两个文件中都要有相应的配置。vite并不依赖于tsc。
