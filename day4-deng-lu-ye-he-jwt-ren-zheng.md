# Day4-登录页和jwt认证

## 前端登录页面实现

使用 Vue3+Vite4+VueRouter+ElementPlus+TailwindCss+AnimateCss 来实现。

### 创建前端项目

> 确保你拥有 nodejs 环境后进行下面操作！

#### 进入 `weblog` 目录中创建前端项目

在目录下执行：

```sh
npm create vue@latest
```

执行过程中，会提示你命名新项目，我们命名为 `weblog-vue3`，以及是否需要开启一些诸如 TypeScript 和测试支持之类的可选功能，这里统一敲击回车键选择 `No` 即可。当你看到命令行中提示 `Done` ， 表示你已经创建好了 `weblog` 前端项目。

#### 添加依赖

```bash
# 进入项目文件夹
cd weblog-vue3
# 安装项目所需依赖
npm install
# 添加vue-router
npm install vue-router
# 添加TailwindCss
npm install -D tailwindcss postcss autoprefixer
# 添加Flowbite组件库
npm install flowbite
# 添加ElementPlus
npm install element-plus --save
# 添加ElementPlus图标库
npm install @element-plus/icons-vue
# 添加自动导入
npm install -D unplugin-vue-components unplugin-auto-import
# 添加AnimateCss
npm install animate.css --save
```

**进行配置**

在添加了上述依赖后，首先打开 `src/main.js` 文件，将内容导入并添加到 Vue `app` 实例中, `main.js` 修改后如下:

```javascript
import '@/assets/main.css'
import 'animate.css'
import * as ElementPlusIconVue from '@element-plus/icons-vue'
import { createApp } from 'vue'
import App from '@/App.vue'
//引入路由
import router from '@/router'

const app = createApp(App)
for (const [key, component] of Object.entries(ElementPlusIconVue)) {
    app.component(key, component)
}
app.use(router).mount('#app')
```

`alias` 是一个用于定义路径别名的配置选项。当你的项目结构变得复杂时，路径别名可以帮助你简化 `import` 或 `require` 语句中的路径。通过 `..` 来指定上一级目录，然后再定位具体路径下。考虑一个大型项目中，我们经常需要这样的引用，当文件层级很深，那么代码可能会像下面这样：

```javascript
import Button from '../../../components/Button.vue';
```

这种相对路径不易于管理和阅读。使用 `alias`，我们可以将路径简化为：

```javascript
import Button from '@/components/Button.vue';
```

这样一来，不仅路径更短、更明确，而且移动文件时无需改动 `import` 路径。

`unplugin-vue-components` 和 `unplugin-auto-import` 这两款插件是实现 `ElementPlus` 组件按需导入的插件。

接下来我们将对这两项进行配置。在项目的根目录中，找到 Vite 的配置文件 `vite.config.js`，编辑它，添加 `alias` 配置并导入插件，代码如下：

```js
import { fileURLToPath, URL } from 'node:url'

import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import AutoImport from 'unplugin-auto-import/vite'
import Components from 'unplugin-vue-components/vite'
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'

// https://vite.dev/config/
export default defineConfig({
  plugins: [
    vue(),
    AutoImport({
      resolvers: [ElementPlusResolver()],
    }),
    Components({
      resolvers: [ElementPlusResolver()],
    }),
  ],
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url))
    }
  }
})

```

接下来对引入的各个依赖进行配置：

1. **vue-router**

在 `src` 目录下，创建 `/pages` 文件夹，此文件夹中统一存放 _页面_ 相关代码，然后 `/pages` 文件夹下，再创建两个文件夹, 分别是：

* `/admin` : 存放管理后台相关页面；
* `/frontend` ：存放前台展示相关页面，如首页、博客详情页等；

在 `/frontend` 文件夹下，创建 `index.vue` 首页文件，代码如下：

```vue
<template>
    <h1>首页</h1>
</template>
```

在 `/src` 目录下，新建 `/router` 路由文件夹，用于存放路由相关代码，并在此文件夹下，新建 `index.js` 文件, 代码如下：

```javascript
import Index from '../pages/frontend/index.vue'
import { createRouter, createWebHashHistory } from 'vue-router'

// 统一在这里声明所有路由
const routes = [
    {
        path: '/', // 路由地址
        component: Index, // 对应组件
        meta: { // meta 信息
            title: 'Weblog 首页' // 页面标题
        }
    }
]

// 创建路由
const router = createRouter({
    // 指定路由的历史管理方式，hash 模式指的是 URL 的路径是通过 hash 符号（#）进行标识
    history: createWebHashHistory(),
    // routes: routes 的缩写
    routes, 
})

// ES6 模块导出语句，它用于将 router 对象导出，以便其他文件可以导入和使用这个对象
export default router
```

上面代码中，我们将首页组件 `index.vue` 导入，并且通过定义 `routes` 数组来统一声明所有的路由，在数组中，我们创建了一个绑定到了根目录 `/` 的路由地址，指向了首页组件。

`<router-view>` 是 Vue Router 的一个核心组件，它是一个功能性组件（functional component）。其主要作用是根据当前的路由（URL）动态地渲染对应的组件，相当于是一个“占位符”，它会自动渲染与当前路径匹配的组件。

* 1、**动态渲染组件**： `<router-view>` 根据当前的 URL，自动渲染与当前路径匹配的组件。
* 2、**嵌套路由**： 在有嵌套路由的情况下，`<router-view>` 可以用于渲染嵌套的子路由组件。

接下来，我们需要在 `app.vue` 中添加该组件，**用于动态渲染路由对应的组件**，再删除 `app.vue` 中的内容后，添加如下代码：

```html
<template>
   <router-view></router-view>
</template>

<script setup>
 
</script>

<style scoped>

</style>
```

2. **TailwindCss**

之前的依赖安装命令会在项目中安装三个依赖，分别为：

* `tailwindcss`：Tailwind CSS 框架本身。
* `postcss`：一个用于转换 CSS 的工具。
* `autoprefixer`：一个 PostCSS 插件，用于自动添加浏览器供应商前缀到 CSS 规则中，确保跨浏览器的兼容性。

同时我们还安装了 `flowbite` 组件库。Flowbite 是一个基于 Tailwind CSS 创建的组件库，旨在帮助开发者快速构建现代、响应式的 Web 应用界面。接下来我们需要进行一些配置以使这些内容生效。

在前端项目根目录执行如下命令, 此命令用于生成 `tailwind.config.js` 和 `postcss.config.js` 配置文件。：

```bash
npx tailwindcss init -p
```

生成后，打开 `tailwind.config.js` 文件，进行如下配置，添加所有模板文件路径，并且引入 `flowbite` 组件库：

```js
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    './index.html',
    './src/**/*.{vue,js,ts,jsx,tsx}',
    "./node_modules/flowbite/**/*.js"
  ],
  theme: {
    extend: {},
  },
  plugins: [
    require('flowbite/plugin')
  ]
}


```

清空 `main.css` 的初始内容，引入 `Tailwind` 样式：

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

body {
  font-family:
    -apple-system-font,
    BlinkMacSystemFont,
    Helvetica Neue,
    PingFang SC,
    Hiragino Sans GB,
    Microsoft YaHei UI,
    Microsoft YaHei,
    Arial,
    sans-serif;
  color: #6b6b6b;
  font-size: 16px;
  background-color: #f8fafc;
  line-height: 1.6;
}


/* 解决样式冲突 */
[type="text"]:focus,
[type="email"]:focus,
[type="url"]:focus,
[type="password"]:focus,
[type="number"]:focus,
[type="date"]:focus,
[type="datetime-local"]:focus,
[type="month"]:focus,
[type="search"]:focus,
[type="tel"]:focus,
[type="time"]:focus,
[type="week"]:focus,
[multiple]:focus,
textarea:focus,
select:focus {
  box-shadow: 0 0 0 1px transparent inset !important;
}

```

### 构建首页框架与登录页

#### 首页框架构建

清空 `src/pages/frontend` 文件夹下 `index.vue` 文件的内容，重写首页，主要是定义了响应式的导航栏，登录按钮加上了 `$router.push('/login')` 的点击跳转路由动作，该路由会在下面定义登录页时添加到 `vue-router` 的配置中：

```html
<template>
<nav class="bg-white border-gray-200 border-b dark:bg-gray-900">
    <div class="max-w-screen-xl flex flex-wrap items-center justify-between mx-auto p-4">
        <a href="/" class="flex items-center">
            <img src="@/assets/day-logo.png" class="h-8 mr-3" alt="Flowbite Logo" />
            <span class="self-center text-2xl font-semibold whitespace-nowrap dark:text-white">小狗的博客</span>
    </a>
        <div class="flex items-center md:order-2">
            <button type="button" data-collapse-toggle="navbar-search" aria-controls="navbar-search"
                    aria-expanded="false"
                    class="md:hidden text-gray-500 dark:text-gray-400 hover:bg-gray-100 dark:hover:bg-gray-700 focus: outline-none focus: ring-4 focus: ring-gray-200 dark:focus:ring-gray-700 rounded-lg text-sm p-2.5 mr-1 ">
                <svg class = "w-5 h-5" aria-hidden = "true" xmlns = "http://www.w3.org/2000/svg" fill = "none"
                     viewBox = "0 0 20 20" >
                    <path stroke = "currentColor" stroke-linecap = "round" stroke-linejoin = "round" stroke-width = "2"
                          d = "m19 19-4-4m0-7A7 7 0 1 1 1 8a7 7 0 0 1 14 0Z" />
    </svg>
                <span class="sr-only"> 搜索 </span>
    </button>
            <div class="relative hidden mr-2 md:block">
                <div class="absolute inset-y-0 left-0 flex items-center pl-3 pointer-events-none">
                    <svg class = "w-4 h-4 text-gray-500 dark: text-gray-400" aria-hidden = "true"
                         xmlns = "http://www.w3.org/2000/svg" fill = "none" viewBox = "0 0 20 20" >
                        < path stroke = "currentColor" stroke-linecap = "round" stroke-linejoin = "round" stroke-width = "2"
                              d = "m19 19-4-4m0-7A7 7 0 1 1 1 8a7 7 0 0 1 14 0Z" />
    </svg>
                    <span class="sr-only"> 搜索图标 </span>
    </div>
                <input type = "text" id = "search-navbar"
                       class =" block w-full p-2 pl-10 text-sm text-gray-900 border border-gray-300 rounded-lg bg-gray-50 focus: ring-blue-500 focus: border-blue-500 dark: bg-gray-700 dark: border-gray-600 dark: placeholder-gray-400 dark: text-white dark:focus:ring-blue-500 dark:focus:border-blue-500 "
                       placeholder = "请输入关键词..." >
    </div>
            <!-- 登录 -->

            <button data-collapse-toggle = "navbar-search" type = "button"
                    class =" inline-flex items-center p-2 w-10 h-10 justify-center text-sm text-gray-500 rounded-lg md: hidden hover: bg-gray-100 focus: outline-none focus: ring-2 focus: ring-gray-200 dark: text-gray-400 dark:hover:bg-gray-700 dark:focus:ring-gray-600 "
                    aria-controls = "navbar-search" aria-expanded = "false" >
                <span class="sr-only"> 打开主菜单 </span>
                <svg class = "w-5 h-5" aria-hidden = "true" xmlns = "http://www.w3.org/2000/svg" fill = "none"
                     viewBox = "0 0 17 14" >
                    <path stroke = "currentColor" stroke-linecap = "round" stroke-linejoin = "round" stroke-width = "2"
                          d = "M1 1h15M1 7h15M1 13h15" />
    </svg>
    </button>
            <div class="text-gray-900 ml-1 mr-1 hover:text-blue-700" @click="$router.push('/login')"> 登录 </div>

    </div>
        <div class="items-center justify-between hidden w-full md:flex md:w-auto md:order-1" id="navbar-search">
            <div class="relative mt-3 md:hidden">
                <div class="absolute inset-y-0 left-0 flex items-center pl-3 pointer-events-none">
                    <svg class = "w-4 h-4 text-gray-500 dark: text-gray-400" aria-hidden = "true"
                         xmlns = "http://www.w3.org/2000/svg" fill = "none" viewBox = "0 0 20 20" >
                        <path stroke = "currentColor" stroke-linecap = "round" stroke-linejoin = "round" stroke-width = "2"
                              d = "m19 19-4-4m0-7A7 7 0 1 1 1 8a7 7 0 0 1 14 0Z" />
    </svg>
    </div>
                <input type = "text" id = "search-navbar"
                       class =" block w-full p-2 pl-10 text-sm text-gray-900 border border-gray-300 rounded-lg bg-gray-50 focus: ring-blue-500 focus: border-blue-500 dark: bg-gray-700 dark: border-gray-600 dark: placeholder-gray-400 dark: text-white dark:focus:ring-blue-500 dark:focus:border-blue-500 "
                       placeholder = "Search..." >
    </div>
            <ul
                class =" flex flex-col p-4 md: p-0 mt-4 font-medium border border-gray-100 rounded-lg bg-gray-50 md: flex-row md: space-x-8 md: mt-0 md: border-0 md: bg-white dark: bg-gray-800 md:dark:bg-gray-900 dark: border-gray-700 ">
                <li>
                    <a href = "#"
                       class =" block py-2 pl-3 pr-4 text-white bg-blue-700 rounded md: bg-transparent md: text-blue-700 md: p-0 md:dark:text-blue-500 "
                       aria-current = "page" > 首页 </a>
    </li>
                <li>
                    <a href = "#"
                       class =" block py-2 pl-3 pr-4 text-gray-900 rounded hover: bg-gray-100 md:hover:bg-transparent md:hover:text-blue-700 md: p-0 md:dark:hover: text-blue-500 dark: text-white dark:hover:bg-gray-700 dark:hover:text-white md:dark:hover: bg-transparent dark: border-gray-700 "> 分类 </a>
    </li>
                <li>
                    <a href = "#"
                       class =" block py-2 pl-3 pr-4 text-gray-900 rounded hover: bg-gray-100 md:hover:bg-transparent md:hover:text-blue-700 md: p-0 dark: text-white md:dark:hover: text-blue-500 dark:hover:bg-gray-700 dark:hover:text-white md:dark:hover: bg-transparent dark: border-gray-700 "> 标签 </a>
    </li>
                <li>
                    <a href = "#"
                       class =" block py-2 pl-3 pr-4 text-gray-900 rounded hover: bg-gray-100 md:hover:bg-transparent md:hover:text-blue-700 md: p-0 dark: text-white md:dark:hover: text-blue-500 dark:hover:bg-gray-700 dark:hover:text-white md:dark:hover: bg-transparent dark: border-gray-700 "> 归档 </a>
    </li>
    </ul>
    </div>
    </div>
    </nav>
</template>

<script setup>
    import { onMounted } from 'vue'
    import { initCollapses } from 'flowbite'

    // 初始化 flowbit 相关组件
    onMounted(() => {
        initCollapses();
    })
</script>
```

#### 登录页构建

在 `src/pages/admin` 文件夹下创建 `login.vue` 文件。登录页构建了一个左右分栏的页面样式，并添加了动画效果，代码如下：

```html
<template>
<div class="grid grid-cols-2 h-full overflow-hidden">
    <!-- 左边栏 -->
    <div
         class = "col-span-2 order-2 p-10 md: col-span-1 md: order-1 bg-gradient-to-br from-slate-900 to-slate-800 animate __animated animate__ fadeInLeft" >
        <div class="flex justify-center items-center h-full flex-col relative">
            <!-- 添加背景动画元素 -->
            <div class="absolute inset-0 opacity-10">
                <div class="floating-squares"> </div>
    </div>

            <h2
                class =" font-bold text-4xl mb-7 text-white tracking-wider animate __animated animate__ fadeInUp animate__delay-1s ">
                Weblog 博客登录
    </h2>
            <p class="text-gray-300 text-center max-w-md animate__animated animate__fadeInUp animate__delay-1s">
                一款由 Spring Boot + Mybaits Plus + Vue 3.2 + Vite 4 开发的前后端分离博客。
    </p>
            <img src = "@/assets/developer.png"
                 class =" w-1/2 mt-8 hover: scale-105 transition-transform duration-300 animate __animated animate__ fadeInUp animate__delay-2s ">
    </div>
    </div>

    <!-- 右边栏 -->
    <div
         class = "col-span-2 order-1 md: col-span-1 md: order-2 bg-gradient-to-br from-teal-50 to-teal-100 animate __animated animate__ fadeInRight" >
        <div class="flex justify-center items-center h-full text-black flex-col p-8">
            <h1
                class = "font-bold text-4xl mb-5 bg-gradient-to-r from-teal-600 to-blue-600 bg-clip-text text-transparent animate __animated animate__ fadeInDown" >
                欢迎回来
    </h1>

            <div
                 class =" flex items-center justify-center mb-7 text-gray-500 space-x-3 animate __animated animate__ fadeInDown animate__delay-1s ">
                <span class="h-[1px] w-16 bg-gradient-to-r from-transparent to-gray-300"> </span>
                <span class="text-sm"> 账号密码登录 </span>
                <span class="h-[1px] w-16 bg-gradient-to-l from-transparent to-gray-300"> </span>
    </div>

            <el-form : model = "form"
                     class =" w-5/6 md: w-2/5 space-y-4 animate __animated animate__ fadeInUp animate__delay-1s ">
                <el-form-item>
                    <el-input size = "large" placeholder = "请输入用户名" : prefix-icon = "User" clearable
                              class = "login-input hover: shadow-lg transition-all duration-300" />
    </el-form-item>
                <el-form-item>
                    <el-input size = "large" type = "password" placeholder = "请输入密码" : prefix-icon = "Lock" clearable
                              class = "login-input hover: shadow-lg transition-all duration-300" show-password />
    </el-form-item>
                <el-form-item>
                    <el-button
                               class = "w-full login-button hover: transform hover: scale-102 transition-all duration-300"
                               size = "large" type = "primary" >
                        登录
    </el-button>
    </el-form-item>
    </el-form>
    </div>
    </div>
    </div>
</template>

<script setup>
    import { User, Lock } from '@element-plus/icons-vue'
    import { ref } from 'vue'
    import 'animate.css'

    const form = ref({})
</script>

<style scoped>
    .login-input : deep(.el-input__wrapper) {
        background: rgba(255, 255, 255, 0.8);
        backdrop-filter: blur(10px);
        border: 2px solid rgba(229, 231, 235, 0.5);
        border-radius: 12px;
        box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
        transition: all 0.3s ease;
    }

    .login-input : deep(.el-input__wrapper.is-focus) {
        border-color: #60A5FA;
        box-shadow: 0 8px 16px -1px rgba(96, 165, 250, 0.2);
    }

    .login-button {
        background: linear-gradient(135deg, #60A5FA 0%, #3B82F6 100%);
        border: none;
        border-radius: 12px;
        font-weight: 600;
        letter-spacing: 0.05em;
        box-shadow: 0 4px 6px -1px rgba(59, 130, 246, 0.2);
    }

    .floating-squares {
        position: absolute;
        width: 100%;
        height: 100%;
        overflow: hidden;
        background: linear-gradient(45deg, rgba(255, 255, 255, 0.1) 25%, transparent 25%),
            linear-gradient(-45deg, rgba(255, 255, 255, 0.1) 25%, transparent 25%),
            linear-gradient(45deg, transparent 75%, rgba(255, 255, 255, 0.1) 75%),
            linear-gradient(-45deg, transparent 75%, rgba(255, 255, 255, 0.1) 75%);
        background-size: 20px 20px;
        animation: animate-squares 20s linear infinite;
    }

    @keyframes animate-squares {
        0% {
            background-position: 0 0, 10px 0, 10px -10px, 0px 10px;
        }

        100% {
            background-position: 40px 40px, 50px 40px, 50px 30px, 40px 50px;
        }
    }
</style>
```

上面样式可以根据自己需要进行调节美化，左右边栏的动画样式可以参考 [AnimateCSS](https://animate.style/) 网站进行修改, 左侧图片出自 [IconFont](https://www.iconfont.cn/) ，可以使用其他图片进行替换，或者直接使用原图 `developer.png` (置于 `src/assets` 文件夹下)：

![developer.png](<.gitbook/assets/developer 1729783297756 2.png>)

接下来配置一下路由，修改 `src\router\index.js` 文件，添加上登录页的路由：

```js
import Index from '@/pages/frontend/index.vue'
import Login from '@/pages/admin/login.vue'
import { createRouter, createWebHistory } from 'vue-router'

//统一在下面声明所有路由
const routes = [
    {
        path: '/',
        component: Index,
        meta: {
            title: 'Weblog 首页'
        }
    },
    {
        path: '/login',
        component: Login,
        meta: {
            title: 'Weblog 登录'
        }
    }
]

//创建路由实例
const router = createRouter({
    history: createWebHistory(),
    routes
})

//导出路由实例
export default router

```

## 数据库连接和JWT登录鉴权

### MyBatis Plus和P6Spy

**MyBatis Plus （简称 MP） 是一款持久层框架**，说白话就是一款操作数据库的框架。它是<mark style="background-color:orange;">一个 MyBatis 的增强工具</mark>，就像 iPhone手机一般都有个 plus 版本一样，它在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生。而 `P6Spy` 组件则用来实现完整的 SQL 打印，在日志中打印SQL语句，能够及时了解每个操作都具体执行了什么，同时通过打印执行耗时，可以发现一些慢SQL，提前做好优化，不过<mark style="background-color:orange;">该组件不适合在生产环境中使用，所以需要在配置文件中进行区分</mark>。

#### 建立数据库

搭建自己的 `mysql` 数据库 `weblog` (此处操作略)。再执行如下建表语句：

```sql
CREATE TABLE `t_user` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT 'id',
  `username` varchar(60) NOT NULL COMMENT '用户名',
  `password` varchar(60) NOT NULL COMMENT '密码',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '最后一次更新时间',
  `is_deleted` tinyint(2) NOT NULL DEFAULT '0' COMMENT '逻辑删除：0：未删除 1：已删除',
  PRIMARY KEY (`id`) USING BTREE,
  UNIQUE KEY `uk_username` (`username`) USING BTREE
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4 COMMENT ='用户表';
```

通过以上操作在 `weblog` 中建立了 `t_user` 表。

#### 整合数据库依赖

在父项目 `weblog-springboot` 的 `pom.xml` 文件中，添加依赖：

```xml
	<!-- 版本号统一管理 -->
    <properties>
        // 省略...
        <mybatis-plus.version> 3.5.2 </mybatis-plus.version>
        <p6spy.version> 3.9.1 </p6spy.version>
    </properties>
    
        <!-- 统一依赖管理 -->
    <dependencyManagement>
        <dependencies>
            // 省略...

            <!-- Mybatis Plus -->
            <dependency>
                <groupId> com.baomidou </groupId>
                <artifactId> mybatis-plus-boot-starter </artifactId>
                <version>${mybatis-plus.version}</version >
            </dependency>
            
            <dependency>
                <groupId> p6spy </groupId>
                <artifactId> p6spy </artifactId>
                <version>${p6spy.version}</version >
            </dependency>


        </dependencies>
    </dependencyManagement>
```

然后，在 `weblog-module-common` 模块的 `pom.xml` 文件中，引入 MP 和 MySQL 依赖：

```xml
		<!-- Mybatis Plus -->
		<dependency>
			<groupId> com.baomidou </groupId>
			<artifactId> mybatis-plus-boot-starter </artifactId>
		</dependency>
		
		<!-- mysql 依赖 -->
		<dependency>
			<groupId> mysql </groupId>
			<artifactId> mysql-connector-java </artifactId>
		</dependency>
```

#### 添加配置

**修改 `application-dev.yml` 配置文件**

```yml
# 应用服务 WEB 访问端口
server:
  port: 8080
spring:
  datasource:
    driver-class-name: com.p6spy.engine.spy.P6SpyDriver # 驱动类使用 p6spy 的代理驱动，用于打印 sql 日志
    url: jdbc:p6spy:mysql://127.0.0.1:3306/weblog?useUnicode = true&characterEncoding = UTF-8&autoReconnect = true&useSSL = false&zeroDateTimeBehavior = convertToNull
    username: root
    password: doga123
    hikari: # HikariCP 连接池配置
      minimum-idle: 5 # 最小空闲连接数
      maximum-pool-size: 20 # 最大连接数
      auto-commit: true # 自动提交
      idle-timeout: 30000 # 连接空闲 30 秒后关闭
      pool-name: Weblog-HikariCP # 连接池名称
      max-lifetime: 1800000 # 连接的生命周期 30 分钟
      connection-timeout: 30000 # 连接超时 30 秒
      connection-test-query: SELECT 1 # 测试连接是否有效的 SQL 语句
```

详细解释一下各个配置的含义和作用：

* `spring.datasource.driver-class-name`: 指定数据库驱动类的完整类名，这里使用的是 MySQL 数据库的驱动类。
* `spring.datasource.url`: 数据库连接的 URL，指向本地的 MySQL 数据库，端口是 3306，数据库名是 `weblog`，同时配置了一系列参数，如使用 Unicode 编码、字符编码为 UTF-8、自动重连、不使用 SSL、对零时间进行转换等。
* `spring.datasource.username`: 数据库用户名，这里使用的是 `root`。
* `spring.datasource.password`: 数据库密码，这里使用的是 `123456`。

数据库链接池这块，我们使用的是 Spring Boot 默认的 HikariCP，它是一个高性能的连接池实现 , 同时，它号称是速度最快的连接池：

* `spring.datasource.hikari.minimum-idle`: Hikari 连接池中最小空闲连接数。
* `spring.datasource.hikari.maximum-pool-size`: Hikari 连接池中允许的最大连接数。
* `spring.datasource.hikari.auto-commit`: 连接是否自动提交事务。
* `spring.datasource.hikari.idle-timeout`: 连接在连接池中闲置的最长时间，超过这个时间会被释放。
* `spring.datasource.hikari.pool-name`: 连接池的名字。
* `spring.datasource.hikari.max-lifetime`: 连接在连接池中的最大存活时间，超过这个时间会被强制关闭。
* `spring.datasource.hikari.connection-timeout`: 获取连接的超时时间。
* `spring.datasource.hikari.connection-test-query`: 用于测试连接是否可用的 SQL 查询，这里使用的是 `SELECT 1`，表示执行这个查询来测试连接。

**修改 `application-prod.yml` 配置文件**

```yml
# 应用服务 WEB 访问端口
server:
  port: 8080
#==== ==== ==== ==== ==== ==== ==== ==== ==== ==== ==== ==== ==== ==== ==== ==== =
# log 日志
#==== ==== ==== ==== ==== ==== ==== ==== ==== ==== ==== ==== ==== ==== ==== ==== =
logging:
  config: classpath: logback-weblog.xml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver # 数据库驱动使用 mysql
    url: jdbc:mysql://127.0.0.1:3306/weblog?useUnicode = true&characterEncoding = UTF-8&autoReconnect = true&useSSL = false&zeroDateTimeBehavior = convertToNull # 数据库连接地址
    username: root
    password: doga123
    hikari:
      minimum-idle: 5
      maximum-pool-size: 20
      auto-commit: true
      idle-timeout: 30000
      pool-name: Weblog-HikariCP
      max-lifetime: 1800000
      connection-timeout: 30000
      connection-test-query: SELECT 1
```

**添加P6Spy配置**

在 `weblog-web` 模块中的 `resources` 目录下添加 `spy.properties` 配置文件：

```properties
modulelist = com.baomidou.mybatisplus.extension.p6spy.MybatisPlusLogFactory
# 自定义日志打印
logMesssageFormat = com.baomidou.mybatisplus.extension.p6spy.P6SpyLogger
#日志输出到控制台
appender = com.baomidou.mybatisplus.extension.p6spy.StdoutLogger
# 使用日志系统记录 sql
# appender = com.baomidou.mybatisplus.extension.p6spy.Slf4jLogger
# 设置 p6spy driver 代理
deregisterdrivers = true
# 取消 JDBC URL 前缀
useprefix = true
# 配置记录 Log 例外, 可去掉的结果集有 error, info, batch, debug, statement, commit, rollback, result, resultset.
excludecategories = info, debug, result, commit, resultest
# 日期格式
dateformat = yyyy-MM-dd HH:mm:ss
# 实际驱动可多个
# driverlist = com.mysql.cj.jdbc.Driver
# 是否开启慢 SQL 记录
outagedetection = true
# 慢 SQL 记录标准 2 秒
outagedetectioninterval = 2
```

#### 测试数据库连接

在 `weblog-module-common` 模块中创建 `config` 包和 `domain` 包，同时在 `domain` 包下创建 `dos` 和 `mapper` 包。在 `config` 包下，新建一个 `MybatisPlusConfig` 配置文件：

```java
package cn.dogalist.weblog.common.config;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@MapperScan("cn.dogalist.weblog.common.domain.mapper") //扫描 mapper 接口
public class MybatisPlusConfig {
}
```

* `@Configuration` : 此注解声明该类为配置类；
*   `@MapperScan` : 扫描 MP 的 `mapper` 接口存放位置。

    > PS: 数据库相关的代码，我们统一放置在 `domain` 这个包中

在 `dos` 包下，新建 `UserDO` 类， <mark style="background-color:orange;">字段和数据库中的字段通过转驼峰的形式对应一一对应起来</mark> ，MP 框架会默认通过这种规则将字段光联在一起：

```java
package cn.dogalist.weblog.common.domain.dos;

import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.Date;


/**
 * @Description: 用户表
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
@TableName("t_user")
public class UserDO {
    @TableId(type = IdType.AUTO)
    private Long id;
    private String username;
    private String password;
    private Date createTime;
    private Date updateTime;
    private Boolean isDeleted;

}
```

在 `mapper` 包中新建 `UserMapper` 类，在该类中构建一个通过用户名在数据库中查找数据条目的方法：

```java
package cn.dogalist.weblog.common.domain.mapper;

import cn.dogalist.weblog.common.domain.dos.UserDO;
import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
public interface UserMapper extends BaseMapper<UserDO> {
    default UserDO findByUsername(String username) {
        // 构建查询条件
        LambdaQueryWrapper<UserDO> queryWrapper = new LambdaQueryWrapper<>();

        queryWrapper.eq(UserDO::getUsername, username);
        return selectOne(queryWrapper);
    }
}

```

在 `weblog-web` 模块中的单元测试类中，新增一个测试方法，代码如下：

```java
@Autowired
private UserMapper userMapper;
@Test
void insertTest() {
    UserDO userDO = UserDO.builder()
            .username("monthwolf")
            .password("123456")
            .createTime(new Date())
            .updateTime(new Date())
            .isDeleted(false)
            .build();
    userMapper.insert(userDO);
}
```

接下来运行该方法，测试数据条目是否成功写入数据库中了，因为我们现在都是在测试环境中运行的，同时可以检查一下配置的 `P6Spy` 是否生效，看看日志中有没有打印出该条SQL语句。

![数据写入](<.gitbook/assets/image 20241025203021721.png>)

![打印SQL](<.gitbook/assets/image 20241025203043077.png>)

## Spring Security 与 JWT 鉴权

### 配置 Spring Security

`Spring Security` 是一个功能强大的安全框架，能够帮助您在应用程序中实现认证（Authentication）和授权（Authorization）功能。在整合 Spring Security 框架之前，我们需要新建一个 `weblog-module-jwt` 子模块(可以参考之前的教程创建)，它主要用于放置 `jwt` 相关的功能代码。创建完成后修改子模块的 `pom.xml` 文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>cn.dogalist</groupId>
    <artifactId>weblog-module-jwt</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>weblog-module-jwt</name>
    <description>weblog-module-jwt (JWT模块，管理用户认证、鉴权)</description>
    <parent>
        <groupId>cn.dogalist</groupId>
        <artifactId>weblog-springboot</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
    <dependencies>
        <!-- 免写冗余的 Java 样板式代码 -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- 单元测试 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!-- Spring Security -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>

        <!-- JWT -->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-api</artifactId>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-impl</artifactId>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-jackson</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- 工具包 -->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>

        <!-- 通用模块 -->
        <dependency>
            <groupId>cn.dogalist</groupId>
            <artifactId>weblog-module-common</artifactId>
        </dependency>

        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
        </dependency>

    
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring</artifactId>
        <version>2.5.6.SEC03</version>
      </dependency>
    </dependencies>
</project>
```

另外，还需在父项目 `weblog-springboot` 的 `pom.xml` 文件中添加 `jwt` 子模块 :

```xml
    <!-- 子模块管理 -->
    <modules>
        // 省略...
        <!-- JWT 模块 -->
        <module>weblog-module-jwt</module>
    </modules>
    
    <!-- 统一依赖管理 -->
    <dependencyManagement>
        <dependencies>
            // 省略...

            <dependency>
                <groupId>cn.dogalist</groupId>
                <artifactId>weblog-module-jwt</artifactId>
                <version>${revision}</version>
            </dependency>

            // 省略...
        </dependencies>
    </dependencyManagement>
```

最后，在 `weblog-module-admin` 模块中，引入 `weblog-module-jwt` 模块和 `Spring Security` 依赖，因为认证、鉴权功能只有 `admin` 后台需要：

```xml
        <dependency>
            <groupId>cn.dogalist</groupId>
            <artifactId>weblog-module-jwt</artifactId>
        </dependency>
		<!-- Spring Security -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
		</dependency>
```

### 自定义配置

要更好地控制 Spring Security 的行为，可以创建一个自定义的 `SecurityConfig` 类，继承自 `WebSecurityConfigurerAdapter`。通过覆盖方法，我们可以配置认证、授权规则、自定义登录页面、注销等。

在 `application-dev.yml`中配置:

```yml
spring:
  // 省略...
  security: # Spring Security 配置
    user: # Spring Security 默认用户，下面内容可自定义
      name: admin 
      password: dog123 
```

在 `weblog-module-admin` 模块中的 `config` 包下，创建一个 `WebSecurityConfig` 配置类，并进行如下自定义配置：

```java
package cn.dogalist.weblog.admin.config;

import cn.dogalist.weblog.jwt.config.JwtAuthenticationSecurityConfig;
import cn.dogalist.weblog.jwt.filter.TokenAuthenticationFilter;
import cn.dogalist.weblog.jwt.handler.RestAuthenticationEntryPoint;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;


/**
 * @Description: Spring Security 配置
 */
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    private JwtAuthenticationSecurityConfig jwtAuthenticationSecurityConfig;
    @Autowired
    private RestAuthenticationEntryPoint authEntryPoint;
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // 禁用CSRF
        http.csrf().disable()
                .formLogin().disable() // 禁用form表单登录
                .apply(jwtAuthenticationSecurityConfig) // 添加JWT过滤器
                .and()
                .authorizeRequests()
                .mvcMatchers("/admin/**").authenticated() // 认证所有以/admin为前缀的URL资源
                .anyRequest().permitAll()// 其他页面放行
                .and()
                .httpBasic().authenticationEntryPoint(authEntryPoint)
                .and()
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS) // 前后端分离，无需创建会话
                .and()
                .addFilterBefore(tokenAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class)
        ;

    }
    /**
     * Token 校验过滤器
     * @return
     */
    @Bean
    public TokenAuthenticationFilter tokenAuthenticationFilter() {
        return new TokenAuthenticationFilter();
    }

}
```

在上面 `configure()` 方法中，主要定义了如下功能：

1.  禁用 CSRF 和表单登录

    禁用 CSRF（跨站请求伪造）保护，因为在前后端分离的架构中，通常使用 JWT 进行身份验证，CSRF 保护不再必要。

    禁用默认的表单登录，改用 JWT 进行身份验证。
2.  应用 JWT 认证配置

    将 `JwtAuthenticationSecurityConfig` 中定义的 JWT 过滤器应用到安全配置中，以处理登录请求和生成 JWT。
3.  URL 访问控制

    保护所有以 `/admin/**` 开头的 URL，要求用户经过身份验证。允许其他所有请求不需要身份验证。
4.  配置 HTTP 基本认证入口点

    使用自定义的 `RestAuthenticationEntryPoint` 处理未认证用户的请求。
5.  会话管理策略

    设置会话管理策略为无状态（`SessionCreationPolicy.STATELESS`），因为在前后端分离的架构中，通常不使用服务器会话。
6.  添加 Token 认证过滤器

    在 `UsernamePasswordAuthenticationFilter` 之前添加 `TokenAuthenticationFilter`，用于从请求中提取和验证 JWT。

实际上我们需要在 `jwt` 模块中实现如下主要类，来使自定义的 `Spring Security` 配置生效：

* **`JwtAuthenticationSecurityConfig`**: 负责配置 JWT 认证的细节，包括成功和失败处理器、用户信息服务和密码编码器。
* **`TokenAuthenticationFilter`**: 负责从请求头中提取 JWT，验证其有效性，并将用户信息存入 `SecurityContextHolder`。
* **`RestAuthenticationEntryPoint`**: 处理未认证用户的请求，返回适当的错误响应。

在 `jwt` 模块中创建如下包，接下进行JWT模块的逐步构建：

![目录结构](<.gitbook/assets/image 20241025231024375.png>)

### 构建JWT认证

> 以下所有内容若无提示都是在 `jwt` 子模块中进行的！

#### 令牌工具类 `JwtTokenHelper`

在 `utils` 包中封装一个 `JwtTokenHelper` 工具类：

```java
package cn.dogalist.weblog.jwt.utils;

import io.jsonwebtoken.*;
import io.jsonwebtoken.security.Keys;
import io.jsonwebtoken.security.SignatureException;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.authentication.BadCredentialsException;
import org.springframework.security.authentication.CredentialsExpiredException;
import org.springframework.stereotype.Component;

import java.security.Key;
import java.time.LocalDateTime;
import java.time.ZoneId;
import java.util.Base64;
import java.util.Date;

/**
 * JWT令牌工具类
 */
@Component
@Slf4j
public class JwtTokenHelper implements InitializingBean {
    /**
     * 签发人
     */
    @Value("${jwt.issuer}")
    private String issuer;
    /**
     * 密钥
     */
    private Key key;
    /**
     * JWT解析
     */
    private JwtParser jwtParser;

    /**
     * 解码配置文件中配置的Base64编码的密钥
     */
    @Value("${jwt.secret}")
    public void setSecret(String base64Key) {
        key = Keys.hmacShaKeyFor(Base64.getDecoder().decode(base64Key));
    }

    /**
     * 初始化JwtParser
     */
    @Override
    public void afterPropertiesSet() throws Exception {
        jwtParser = Jwts.parserBuilder().requireIssuer(issuer)
                .setSigningKey(key)
                .setAllowedClockSkewSeconds(10)
                .build();
    }

    /**
     * 生成token
     * 
     * @param username
     * @return token
     */
    public String generateToken(String username) {
        LocalDateTime now = LocalDateTime.now();
        // 一小时后失效
        LocalDateTime expiration = now.plusHours(1);
        return Jwts.builder().setSubject(username)
                .setIssuer(issuer)
                .setIssuedAt(Date.from(now.atZone(ZoneId.systemDefault()).toInstant()))
                .setExpiration(Date.from(expiration.atZone(ZoneId.systemDefault()).toInstant()))
                .signWith(key)
                .compact();
    }

    /**
     * 解析token
     * 
     * @param token
     * @return
     */
    public Jws<Claims> parseToken(String token) {
        try {
            return jwtParser.parseClaimsJws(token);
        } catch (SignatureException | MalformedJwtException e) {
            throw new BadCredentialsException("token不可用");
        } catch (ExpiredJwtException e) {
            throw new CredentialsExpiredException("token已过期");
        }
    }

    /**
     * 校检token是否可用
     * @param token
     * @return
     */
    public void validateToken(String token) {
        // parseClaimsJws方法会校检token是否可用，如果不可用会抛出这几种异常：
        // SignatureException、MalformedJwtException、UnsupportedJwtException、IllegalArgumentException、ExpiredJwtException
        jwtParser.parseClaimsJws(token);
    }
    /**
     * 解析token中的用户名
     * @param token
     * @return
     */
    public String getUsernameFromToken(String token) {
        try {
            // 解析token字段
            Claims claims = jwtParser.parseClaimsJws(token).getBody();
            return claims.getSubject(); // 获取用户名，即subject字段
        } catch (Exception e) {
            log.error("解析token时发生异常: {}", e.getMessage(), e);
        }
        return null;
    }

    /**
     * 生成一个Base64编码的密钥
     * 
     * @Return String
     */
    private static String generatebase64Key() {
        // 生成安全密匙
        Key secretKey = Keys.secretKeyFor(SignatureAlgorithm.HS512);
        // 将密匙转为base64
        return Base64.getEncoder().encodeToString(secretKey.getEncoded());
    }

    public static void main(String[] args) {
        System.out.println("key:" + generatebase64Key());
    }
}
```

通过运行该工具类的 `main` 方法，我们可以得到一个 base64 编码的密匙，将密匙存放到我们的 `application.yml` 配置中，同时设置 token 的签发人（可以自行填写）：

```yml
# ...其他配置省略
jwt:
  # 签发人（自行填写）
  issuer: xxx
  # 秘钥(填写自己生成的密匙)
  secret: xxxxxx
```

#### 密码加密类 `PasswordEncoderConfig`

使用明文密码很容易受到工具，所以我们需要用密码加密算法来保护用户密码。

在 `config` 包中创建 `PasswordEncoderConfig` 配置类：

```java
package cn.dogalist.weblog.jwt.config;

import org.springframework.context.annotation.Bean;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Component;

/**
 * 密码加密配置类
 */
@Component
public class PasswordEncoderConfig {
    @Bean
    public PasswordEncoder passwordEncoder() {
        // 使用 BCrypt 加密
        return new BCryptPasswordEncoder();
    }

    // 测试BCrypt加密密码
    public static void main(String[] args) {
        BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
        System.out.println(encoder.encode("dog123")); // 加密自己配置的Spring Security用户密码
    }

}
```

`PasswordEncoder` 接口是 Spring Security 提供的密码加密接口，它定义了密码加密和密码验证的方法。通过实现这个接口，您可以将密码加密为不可逆的哈希值，以及在验证密码时对比哈希值。

上述代码中，我们初始化了一个 `PasswordEncoder` 接口的具体实现类 `BCryptPasswordEncoder`。`BCryptPasswordEncoder` 是 Spring Security 提供的密码加密器的一种实现，使用 BCrypt 算法对密码进行加密。BCrypt 是一种安全且适合密码存储的哈希算法，它在进行哈希时会自动加入“盐”，增加密码的安全性。通过运行该工具类的 `main` 方法，我们就得到了加密后的用户密码，请复制得到的加密密码，接下来将会使用到。

#### 用户详细服务类 `UserDetailServiceImpl`

编辑 `UserMapper` 接口

在 `service` 包中创建 `UserDetailServiceImpl` 实现类：

```java
package cn.dogalist.weblog.jwt.service;

import cn.dogalist.weblog.common.domain.dos.UserDO;
import cn.dogalist.weblog.common.domain.mapper.UserMapper;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

import java.util.Objects;

/**
 * 用户信息获取类
 */
@Service
@Slf4j
public class UserDetailServiceImpl implements UserDetailsService {
    @Autowired
    private UserMapper userMapper;
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {

        // 从数据库中查询
        UserDO userDO = userMapper.findByUsername(username);

        // 判断用户是否存在
        if (Objects.isNull(userDO)) {
            throw new UsernameNotFoundException("该用户不存在");
        }
        
        return User.withUsername(userDO.getUsername())
                .password(userDO.getPassword())
                .authorities("ADMIN")
                .build();
    }

}

```

该类是`UserDetailsService` 接口的实现。`UserDetailsService` 是 Spring Security 提供的接口，用于从应用程序的数据源（如数据库、LDAP、内存等）中加载用户信息。它是一个用于将用户详情加载到 Spring Security 的中心机制。`UserDetailsService` 主要负责两项工作：

1. **加载用户信息：** 从数据源中加载用户的用户名、密码和角色等信息。
2. **创建 UserDetails 对象：** 根据加载的用户信息，创建一个 Spring Security 所需的 `UserDetails` 对象，包含用户名、密码、角色和权限等。

在上面代码中，我们实现了 `UserDetailsService` 接口，并重写了 `loadUserByUsername()` 方法，该方法用于根据用户名加载用户信息的逻辑 ，需要从数据库中查询。在数据库表`t_user`中执行如下语句添加用户(请根据自己的配置文件定义修改用户名，密码需要是上一步加密后生成的密码)：

```sql
INSERT INTO `weblog`.`t_user` (`username`, `password`, `create_time`, `update_time`, `is_deleted`) VALUES ('admin', '$2a$10$WmLxFbhHbf3hltaTSnNdOuxRfcdgG7qkDUI5rbhS/sBnE.JgcYPeS', now(), now(), 0);

```

#### 自定义认证过滤器 `JwtAuthenticationFilter`

在 `exception` 包下新建 `UsernameOrPasswordNullException` 类：

```java
package cn.dogalist.weblog.jwt.exception;

import org.springframework.security.core.AuthenticationException;

/**
 * 用户名或密码为空异常
 * 注意，需继承自 AuthenticationException，只有该类型异常，才能被后续自定义的认证失败处理器捕获到。
 */
public class UsernameOrPasswordNullException extends AuthenticationException {
    // 构造函数，传入异常信息
    public UsernameOrPasswordNullException(String msg) {
        super(msg);
    }

    // 构造函数，传入异常信息和异常
    public UsernameOrPasswordNullException(String msg, Throwable t) {
        super(msg, t);
    }
}
```

然后在 `filter` 包下新建 `JwtAuthenticationFilter` 类：

```java
package cn.dogalist.weblog.jwt.filter;

import cn.dogalist.weblog.jwt.exception.UsernameOrPasswordNullException;
import com.baomidou.mybatisplus.core.toolkit.StringUtils;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.authentication.AbstractAuthenticationProcessingFilter;
import org.springframework.security.web.util.matcher.AntPathRequestMatcher;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Objects;

public class JwtAuthenticationFilter extends AbstractAuthenticationProcessingFilter {
    /**
     * 指定用户登录的访问地址
     */
    public JwtAuthenticationFilter() {
        super(new AntPathRequestMatcher("/login", "POST"));
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException, IOException, ServletException {
        ObjectMapper mapper = new ObjectMapper();
        // 解析提交的json数据
        JsonNode jsonNode = mapper.readTree(request.getInputStream());
        JsonNode usernameNode = jsonNode.get("username");
        JsonNode passwordNode = jsonNode.get("password");

        // 校检用户名密码是否为空
        if (Objects.isNull(usernameNode) || Objects.isNull(passwordNode) ||  StringUtils.isBlank(usernameNode.textValue()) || StringUtils.isBlank(passwordNode.textValue())) {
            throw new UsernameOrPasswordNullException("用户名或密码不能为空");
        }
        String username = usernameNode.textValue();
        String password = passwordNode.textValue();

        // 封装成UsernamePasswordAuthenticationToken
        UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(username, password);
        // 返回Authentication
        return getAuthenticationManager().authenticate(authenticationToken);
    }

}
```

过滤器继承了 `AbstractAuthenticationProcessingFilter`，用于处理 JWT（JSON Web Token）的用户身份验证过程。在构造函数中，调用了父类 `AbstractAuthenticationProcessingFilter` 的构造函数，通过 `AntPathRequestMatcher` 指定了处理用户登录的访问地址。这意味着当请求路径匹配 `/login` 并且请求方法为 `POST` 时，该过滤器将被触发。

`attemptAuthentication()` 方法用于实现用户身份验证的具体逻辑。首先，我们解析了提交的 JSON 数据，并获取了用户名、密码，校验是否为空，若不为空，则将它们封装到 `UsernamePasswordAuthenticationToken` 中。最后，使用 `getAuthenticationManager().authenticate()` 来触发 Spring Security 的身份验证管理器执行实际的身份验证过程，然后返回身份验证结果。

#### 自定义认证处理器（成功&失败）

在 Spring Security 中，`AuthenticationFailureHandler` 和 `AuthenticationSuccessHandler` 是用于处理身份验证失败和成功的接口。它们允许您在用户身份验证过程中自定义响应，以便更好地控制和定制用户体验。

在 `model` 包下新建`LoginRspVO` 类。此类是登录的响参类，`VO` (View Object) 表示和视图相关的实体类，`rsp` 是 `response` 的缩写，表示返参，对应的 `req` 是 `request` 的缩写，表示请求。代码如下：

```java
package cn.dogalist.weblog.jwt.model;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

/**
 * 登录响应VO
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class LoginRspVO {
    /**
     * token
     */
    private String token;
    /**
     * 用户名
     */
    private String username;

    /**
     * 过期时间
     */
    private Long expireTime;
}
```

同时为了在过滤器中方便的返回 JSON 参数，我们需要封装一个工具类 `ResultUtil`， 放置在 `/utils` 包下，代码如下：

```java
package cn.dogalist.weblog.jwt.utils;

import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.http.HttpStatus;

import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

/**
 * 结果工具类
 */
public class ResultUtil {
    /**
     * 成功响应
     * 
     * @param response
     * @param result
     * @throws IOException
     */
    public static void ok(HttpServletResponse response, Object result) throws IOException {
        response.setCharacterEncoding("UTF-8");
        response.setStatus(HttpStatus.OK.value()); // 200
        response.setContentType("application/json");

        // 输出响应
        PrintWriter writer = response.getWriter();
        ObjectMapper mapper = new ObjectMapper();
        // 忽略为null的字段
        mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
        // 将结果转换为json字符串
        writer.write(mapper.writeValueAsString(result));
        writer.flush();
        writer.close();
    }

    /**
     * 失败响应
     * 
     * @param response
     * @param result
     * @throws IOException
     */
    public static void fail(HttpServletResponse response, Object result) throws IOException {
        response.setCharacterEncoding("UTF-8");
        response.setStatus(HttpStatus.OK.value());
        response.setContentType("application/json");
        PrintWriter writer = response.getWriter();
        ObjectMapper mapper = new ObjectMapper();
        mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
        writer.write(mapper.writeValueAsString(result));
        writer.flush();
        writer.close();
    }

    /**
     * 失败响应
     * 
     * @param response
     * @param status
     * @param result
     * @throws IOException
     */
    public static void fail(HttpServletResponse response, int status, Object result) throws IOException {
        response.setCharacterEncoding("UTF-8");
        response.setStatus(status);
        response.setContentType("application/json");
        PrintWriter writer = response.getWriter();
        ObjectMapper mapper = new ObjectMapper();
        mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
        writer.write(mapper.writeValueAsString(result));
        writer.flush();
        writer.close();
    }

}
```

编辑 `weblog-module-common` 模块中的 `ResponseCodeEnum` 枚举类，添加用户异常类型的响应码：

```java
// 其他略
// ----------- 用户异常状态码 -----------
LOGIN_FAIL("20000", "登录失败"),
USERNAME_OR_PWD_ERROR("20001", "用户名或密码错误"), 
UNAUTHORIZED("20002","无权限访问，请先登录！" );
```

在 `handler` 包下创建 `RestAuthenticationSuccessHandler` 类，用于处理认证成功的数据返回：

```java
package cn.dogalist.weblog.jwt.handler;

import cn.dogalist.weblog.common.utils.Response;
import cn.dogalist.weblog.jwt.model.LoginRspVO;
import cn.dogalist.weblog.jwt.utils.JwtTokenHelper;
import cn.dogalist.weblog.jwt.utils.ResultUtil;
import lombok.extern.slf4j.Slf4j;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.web.authentication.AuthenticationSuccessHandler;
import org.springframework.stereotype.Component;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * 认证成功处理器
 */
@Component
@Slf4j
public class RestAuthenticationSuccessHandler implements AuthenticationSuccessHandler {
    @Autowired
    private JwtTokenHelper jwtTokenHelper;

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response,
            Authentication authentication) throws IOException, ServletException {
        // 从 authentication 对象中获取用户的 UserDetails 实例，这里是获取用户的用户名
        UserDetails userDetails = (UserDetails) authentication.getPrincipal();
        // 生成token
        String username = userDetails.getUsername();
        String token = jwtTokenHelper.generateToken(username);
        // 从token获取过期时间
        Long expireTime = jwtTokenHelper.parseToken(token).getBody().getExpiration().getTime();

        // 返回token
        LoginRspVO loginRspVO = LoginRspVO.builder().token(token).expireTime(expireTime).username(username).build();
        log.info("用户{}登录成功，token:{}", username, token);
        ResultUtil.ok(response, Response.success(loginRspVO));
    }
}
```

该自定义的认证成功处理器首先从 `authentication` 对象中获取用户的 `UserDetails` 实例，这里是主要是获取用户的用户名，然后通过用户名生成 `Token` 令牌，最后返回数据。

再在 `/handler` 包下，创建 `RestAuthenticationFailureHandler` 认证失败处理器：

```java
package cn.dogalist.weblog.jwt.handler;

import cn.dogalist.weblog.common.enums.ResponseCodeEnum;
import cn.dogalist.weblog.common.utils.Response;
import cn.dogalist.weblog.jwt.exception.UsernameOrPasswordNullException;
import cn.dogalist.weblog.jwt.utils.ResultUtil;
import lombok.extern.slf4j.Slf4j;
import org.springframework.security.authentication.BadCredentialsException;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.authentication.AuthenticationFailureHandler;
import org.springframework.stereotype.Component;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * 认证失败处理器
 */
@Component
@Slf4j
public class RestAuthenticationFailureHandler implements AuthenticationFailureHandler {
    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response,
            AuthenticationException exception) throws IOException, ServletException {
        log.warn("Authentication failed: " + exception);
        if (exception instanceof UsernameOrPasswordNullException) {
            // 用户名或密码为空
            ResultUtil.fail(response, Response.fail(exception.getMessage()));
        } else if (exception instanceof BadCredentialsException) {
            // 用户名或密码错误
            ResultUtil.fail(response, Response.fail(ResponseCodeEnum.USERNAME_OR_PWD_ERROR));
        }
        // 登录失败
        ResultUtil.fail(response, Response.fail(ResponseCodeEnum.LOGIN_FAIL));
    }

}
```

这个类实现了 `AuthenticationFailureHandler` 接口，主要用于处理用户认证失败的情况。

当用户登录失败时，`onAuthenticationFailure` 方法会被调用。这个方法接收三个参数：`HttpServletRequest`、`HttpServletResponse` 和 `AuthenticationException`。在方法内部，首先通过日志记录认证失败的详细信息。接着，根据异常的类型，返回不同的错误信息。如果异常是 `UsernameOrPasswordNullException`，则表示用户名或密码为空，使用 `ResultUtil.fail` 方法返回相应的错误信息。如果异常是 `BadCredentialsException`，则表示用户名或密码错误，同样使用 `ResultUtil.fail` 返回错误信息。最后，若认证失败的原因不属于上述两种情况，则返回一个通用的登录失败信息。

#### 自定义token校检过滤器 `TokenAuthenticationFilter`

在建立过滤器之前，我们需要在 `/handler` 包下，新增 `RestAuthenticationEntryPoint` 处理器，它专门用来处理当用户未登录时，访问受保护的资源的情况：

```java
package cn.dogalist.weblog.jwt.handler;

import cn.dogalist.weblog.common.enums.ResponseCodeEnum;
import cn.dogalist.weblog.common.utils.Response;
import cn.dogalist.weblog.jwt.utils.ResultUtil;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.security.authentication.InsufficientAuthenticationException;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.AuthenticationEntryPoint;
import org.springframework.stereotype.Component;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 *
 * @Description: 未登录访问受保护资源处理器
 */
@Slf4j
@Component
public class RestAuthenticationEntryPoint implements AuthenticationEntryPoint {

    /**
     * 覆写认证失败处理器
     * @param request
     * @param response
     * @param authException
     * @throws IOException
     * @throws ServletException
     */
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
        log.warn("用户未登录访问受保护的资源", authException);
        if (authException instanceof InsufficientAuthenticationException) {
            ResultUtil.fail(response, HttpStatus.UNAUTHORIZED.value(), Response.fail(ResponseCodeEnum.UNAUTHORIZED));
        }
        ResultUtil.fail(response, HttpStatus.UNAUTHORIZED.value(), Response.fail(authException.getMessage()));
    }

}
```

当请求受保护的接口且请求头中未携带 `token` 时，我们收到的异常类型为 `InsufficientAuthenticationException`，请求的 `HTTP` 响应码为 401, 这就是未授权的异常类型。因此将这个类型的异常单独拎出来处理。覆写的方法主要实现当校验到 Token 异常时，进行捕获并返回不同的提示信息。

在 `/filter` 包下，创建 `TokenAuthenticationFilter` 过滤器，专用于校检 `token` 令牌：

```java
package cn.dogalist.weblog.jwt.filter;

import cn.dogalist.weblog.jwt.utils.JwtTokenHelper;
import io.jsonwebtoken.ExpiredJwtException;
import io.jsonwebtoken.MalformedJwtException;
import io.jsonwebtoken.UnsupportedJwtException;
import io.jsonwebtoken.security.SignatureException;
import org.apache.commons.lang3.StringUtils;
import org.checkerframework.checker.units.qual.A;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.authentication.AuthenticationServiceException;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.web.AuthenticationEntryPoint;
import org.springframework.security.web.authentication.WebAuthenticationDetails;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import lombok.extern.slf4j.Slf4j;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Objects;

/**
 * 令牌认证过滤器
 */
@Slf4j
public class TokenAuthenticationFilter extends OncePerRequestFilter {
    @Autowired
    private JwtTokenHelper jwtTokenHelper;
    @Autowired
    private UserDetailsService userDetailsService;
    @Autowired
    private AuthenticationEntryPoint authenticationEntryPoint;

    /**
     * 认证过滤器
     * @param request
     * @param response
     * @param filterChain
     * @throws ServletException
     * @throws IOException
     */
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        // 从请求头中获取Authorization字段
        String authorization = request.getHeader("Authorization");
        // 判断值是否以Bearer开头
        if (authorization != null && authorization.startsWith("Bearer ")) {
            // 获取token
            String token = authorization.substring(7);
            log.info("token: {}", token);
            // 判断token是否为空
            if (StringUtils.isNotBlank(token)) {
                // 校检token是否可用
                try {
                    jwtTokenHelper.validateToken(token);
                } catch (SignatureException | MalformedJwtException | UnsupportedJwtException |
                         IllegalArgumentException e) {
                    authenticationEntryPoint.commence(request, response, new AuthenticationServiceException("token不可用"));
                    return;
                } catch (ExpiredJwtException e) {
                    authenticationEntryPoint.commence(request, response, new AuthenticationServiceException("token已过期"));
                    return;
                }
                // 从token中获取用户名
                String username = jwtTokenHelper.getUsernameFromToken(token);
                // 判断用户名是否为空且用户是否认证
                if (StringUtils.isNotBlank(username) && Objects.isNull(SecurityContextHolder.getContext().getAuthentication())) {
                    // 从数据库中获取用户信息
                    UserDetails userDetails = userDetailsService.loadUserByUsername(username);

                    // 将用户信息放入 authentication
                    UsernamePasswordAuthenticationToken authentication = new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
                    // 将请求信息放入 authentication
                    authentication.setDetails(new WebAuthenticationDetails(request));
                    // 将 authentication 放入上下文
                    SecurityContextHolder.getContext().setAuthentication(authentication);
                }
            }
        }
        // 放行请求
        filterChain.doFilter(request, response);
    }

}
```

这一步自定义了一个 Spring Security 过滤器 `TokenAuthenticationFilter`，它继承了 `OncePerRequestFilter`，确保每个请求只被过滤一次。在重写的 `doFilterInternal()` 方法中来定义过滤器处理逻辑，首先，从请求头中获取 `key` 为 `Authorization` 的值，判断是否以 Bearer 开头，若是，截取出 `Token`, 对其进行解析，并对可能抛出的异常做出不同的返参。最后，我们获取了用户详情信息，将用户信息存入 `authentication`，方便后续进行校验，同时将 `authentication` 存入 `ThreadLocal` 中，方便后面方便的获取用户信息，然后在处理完成并且没有出错之后放行请求。

你可能注意到在上面我们实现了处理器类 `RestAuthenticationEntryPoint` ，但上面 `@Autowired` 注解的是该类实现的接口 `AuthenticationEntryPoint` 。在 Spring 框架中，`@Autowired` 注解用于自动装配依赖项。当你在类中声明一个带有 `@Autowired` 注解的字段时，Spring 容器会在启动时自动查找与该字段类型匹配的 Bean，并将其注入到该字段中。在该项目中，因为我们之定义了一个 `AuthenticationEntryPoint` 类型的 Bean `RestAuthenticationEntryPoint` ，可以直接使用 `AuthenticationEntryPoint` 。但如果有多个实现，则需要使用 `@Qualifier("restAuthenticationEntryPoint")` 进行指定。

#### 权限不足处理器 `RestAccessDeniedHandler`

在 `/handler` 包下，另外新增 `RestAccessDeniedHandler` 处理器，它主要用于处理当用户登录成功时，访问受保护的资源，但是权限不够的情况：

```java
package cn.dogalist.weblog.jwt.handler;

import lombok.extern.slf4j.Slf4j;
import org.springframework.security.access.AccessDeniedException;
import org.springframework.security.web.access.AccessDeniedHandler;
import org.springframework.stereotype.Component;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Slf4j
@Component
public class RestAccessDeniedHandler implements AccessDeniedHandler {
    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException, ServletException {
        log.warn("登录成功访问受保护的资源，但是权限不足: ", accessDeniedException);
        // todo 预留，后面引入多角色时会用到
    }
}
```

该处理器目前并不会用到，为了后续可能引入多角色时，预留该类方便拓展使用。

#### 自定义 JWT 认证配置 `JwtAuthenticationSecurityConfig`

在 `/config` 包下新建 `JwtAuthenticationSecurityConfig`, 代码如下：

```java
package cn.dogalist.weblog.jwt.config;

import cn.dogalist.weblog.jwt.filter.JwtAuthenticationFilter;
import cn.dogalist.weblog.jwt.handler.RestAuthenticationFailureHandler;
import cn.dogalist.weblog.jwt.handler.RestAuthenticationSuccessHandler;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
import org.springframework.security.config.annotation.SecurityConfigurerAdapter;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.DefaultSecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

/**
 * JWT认证配置类
 */
@Configuration
public class JwtAuthenticationSecurityConfig
        extends SecurityConfigurerAdapter<DefaultSecurityFilterChain, HttpSecurity> {
    // 用Autowired注解注入成功和失败的处理器、密码加密方式、用户信息获取类
    @Autowired
    private RestAuthenticationSuccessHandler restAuthenticationSuccessHandler;

    @Autowired
    private RestAuthenticationFailureHandler restAuthenticationFailureHandler;

    @Autowired
    private PasswordEncoder passwordEncoder;

    @Autowired
    private UserDetailsService userDetailsService;

    @Override
    public void configure(HttpSecurity http) throws Exception {
        // 自定义用于JWT的过滤器
        JwtAuthenticationFilter filter = new JwtAuthenticationFilter();
        filter.setAuthenticationManager(http.getSharedObject(AuthenticationManager.class));

        // 设置登录认证对应处理类
        filter.setAuthenticationSuccessHandler(restAuthenticationSuccessHandler);
        filter.setAuthenticationFailureHandler(restAuthenticationFailureHandler);

        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();

        // 设置密码加密方式
        provider.setPasswordEncoder(passwordEncoder);

        // 设置用户信息获取类
        provider.setUserDetailsService(userDetailsService);
        http.authenticationProvider(provider);
        http.addFilterBefore(filter, UsernamePasswordAuthenticationFilter.class);

    }

}
```

上述代码是一个 Spring Security 配置类，用于配置 JWT（JSON Web Token）的身份验证机制。它继承了 Spring Security 的 `SecurityConfigurerAdapter` 类，用于在 Spring Security 配置中添加自定义的认证过滤器和提供者。通过重写 `configure()` 方法，我们将之前写好过滤器、认证成功、失败处理器，以及加密算法整合到了 `httpSecurity` 中。

### 测试

使用设置的用户名和密码（明文）请求 `\login` 登录接口，得到`token`值。

添加新的测试API:

```java
// 测试admin登录安全
@PostMapping("/admin/test4")
@ApiOperationLog(description = "测试")
@ApiOperation(value = "测试admin登录安全")
public Response test4(@RequestBody @Validated User user) {
    // 打印入参
    log.info(JsonUtil.toJsonString(user));

    // 设置三种日期字段值
    user.setCreateTime(LocalDateTime.now());
    user.setUpdateDate(LocalDate.now());
    user.setTime(LocalTime.now());

    return Response.success(user);
}
```

在不设置`Authorization`请求头时请求上面接口，设置`Authorization`请求头（值为`Bearer` + token值），以及输入错误的请求头后得到测试结果返回正常：

![未设置token](<.gitbook/assets/image 20241026011530221.png>)

![token正确](<.gitbook/assets/image 20241026011607760.png>)

![token错误](<.gitbook/assets/image 20241026011551459.png>)
