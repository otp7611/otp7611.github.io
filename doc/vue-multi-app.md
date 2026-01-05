把多个应用整合到同一个vue工程

vue是一个非常优秀的前端框架。如果你想写漂亮的前端界面，那vue是首选项。

# 为什么需要多个应用整合到同一个vue工程

vue是渐进式框架，你可以在现有的网页中嵌入使用它。指定一个div元素作为挂载点，把vue实例挂载到这个div元素上，就可以使用vue的功能了。这个过程很简单。但是当你有多个应用时，每个应用都按教程指导创建多个vue工程，然后还要用源码管理工具管理这些应用工程时，就会觉得是多么无聊的事情。于是，把多个应用整合到同一个vue工程的想法就产生了。

# 如何实现

## 按vue官方文档创建vue应用

```bash
npm create vue@latest
```

在创建工程过程中把Add TypeScript和Add Vue Router for Single Page Application development这两个选中。表示使用TypeScript和Router的。这两个功能很实用。这里先把这两个功能选上，免得后面修改vue工程结构。

## 构建工程

```bash
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

## 在index.html中指定搭载点

```html
<div id="app"></div>
<script type="module" src="/src/main.ts"></script>
```

## 在main.ts依据不同的url地址创建vue实例

```typescript
import App1 from "/src/app1/App.vue"
import app1router from "/src/app1/router"
import App2 from "/src/app2/App.vue"
import app2router from "/src/app2/router"

async function createAndMountApp(pApp, pRouter) {
    const app = createApp(pApp)
    app.use(pRouter)
    await pRouter.isReady() // wait for router become ready.
    let rootNode = app.mount("#app")
}

async function start() {
    let location = new URL(window.location.href)
    if (location.pathname.startsWith("/app1")) {
        await createAndMountApp(App1, app1router)
    } else if (location.pathname.startsWith("/app2")) {
        await createAndMountApp(App2, app2router)
    }
}
```

rootNode是vue根dom节点。

# 最终效果

按上操作，可以通过不同的url进入不同的应用界面。
