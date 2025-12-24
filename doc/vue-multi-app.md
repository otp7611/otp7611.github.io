把多个应用整合到同一个vue工程

vue是一个非常优秀的前端框架。如果你想写前端漂亮的界面，那vue是首选项。

# 为什么需要多个应用整合到同一个vue工程

vue是渐进式框架，你可以在现有的网页中嵌入使用它。指定一个div元素作为挂载点，把vue实例挂载到这个div元素上，就可以使用vue的功能了。这个过程很简单，但是当你有多个应用时，每个应用都按教程创建多个vue工程，然后还要用源码管理工具对管理这些应用工程时，就会觉得是多么无聊的事情。于是，把多个应用整合到同一个vue工程的想法就产生了。

# 如何实现

## 按vue官方文档创建vue应用

```bash
npm create vue@latest
```

在创建工程过程中把Add TypeScript和Add Vue Router for Single Page Application development这两个选中。表示使用TypeScript和Router的。这两个功能很实用，使用vue写过一段时间后，你就会自然产生对这两个特性的需求了。这里先把这两个功能选上，免得后面修改vue工程结构。

## 构建工程

```
npm run build
```

在配置文件package.json中scripts->build你会发现调用的是vite build。vite使用配置文件vite.config.js.在vite.config.js中，默认的打包入口点是index.html.你也可以指定其它的入口点。

```json
export default defineConfig({
  build: {
    rollupOptions: {
      input: {
        main: 'index.html'
      }
    }
  }
})
```

