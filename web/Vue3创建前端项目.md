# **Vue3创建前端项目**

## *1.创建、运行Vite项目*


完成Node.js安装以及npm包管理器配置后，便可以使用npm包管理器安装Vite。npm并没有图形可视化界面，同样也是在cmd终端中使用命令行操作，。输入以下指令以安装，详见[Vite中文文档](https://cn.vitejs.dev/guide/)：

**注意：以下命令的输入意味着会在终端当前所在目录下创建项目文件夹，请确保终端所在的目录是你心仪的项目文件夹，以下是Windows cmd基础命令：**

```css
#输入路径或文件名时键入开头按下Tab可实现自动补全
cd 路径 		#使终端所在的目录移动至指定路径
cd ../		 #返回上一级目录
dir 		 #显示当前所在目录下的所有文件和目录
```

使用 NPM:

```css
npm create vite@latest
```

使用 Yarn:

```css
yarn create vite
```

使用 PNPM:

```css
pnpm create vite
```
### (1).项目初始化

执行上述步骤后，命令行中会示意交互安装，如图所示。

**操作方式**：命名选项可直接键入字符，回车确认。选择式选项通过上下左右方向键选择，回车确认。
<img width="683" height="311" alt="image" src="https://github.com/user-attachments/assets/e2635a63-8aeb-401f-bd40-7f885b0c0b5c" />

### (2).自动化下载依赖，起洞项目

打开资源管理器，来到终端执行上述命令所在的目录，即项目目录。会发现有一个文件夹，名字就是你刚刚键入的项目名称，那便是一整个项目的文件夹了。若你还没有关闭终端，你可以使用以下命令使终端进入项目文件夹。Win11可以直接在资源管理器中进入该文件夹右键空白处选择’在终端中打开’。

```css
cd Example-1 #Example-1是我示例输入的项目名
```

进入项目文件夹后，键入以下指令以安装项目必须依赖：

```css
npm i 
```

该命令全称为 `npm install` 执行改命令会使npm在终端当前目录下自动寻找package.json文件并自动下载其中包含的依赖信息。大家可以在项目文件夹中找到该文件。

执行无误后会输出：

```css
D:\Coding\Ex\Vite\Example-1>npm i

added 29 packages in 2s
```

## *2.Tailwindcss的设置*

在终端输入

```bash
npm install -D tailwindcss postcss autoprefixer
//上面这个有时候可能会出错，不行可以使用下面这个
npm install -D tailwindcss@3.4.13 postcss autoprefixer
npx tailwindcss init -p
```

在tailwind.config.js文件中输入

```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{vue,js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

在style.css文件中输入

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```
**补充**：可能会存在**安装好之后用不了**的情况--即**没有被渲染**
我们可以检查我们是**否有`postcss.config.js`**，没有的话我们需要手动安装
```js
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

跳转到`idea`
<img width="729" height="558" alt="image" src="https://github.com/user-attachments/assets/50c0b1db-b82c-4f74-814a-c42627f13456" />

**一切准备好后就可以使用Tailwindcss**
## ***4.组件库安装***

1.[Element](https://element-plus.org/zh-CN/guide/quickstart.html)

在终端中输入

```css
 npm install element-plus --save
```

2.[Antdv](https://www.antdv.com/docs/vue/getting-started-cn)

先在终端中输入

```css
npm i --save ant-design-vue@4.x
```

再在`main.js`中插入

```css
import Antd from 'ant-design-vue';
import 'ant-design-vue/dist/reset.css';
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'
```

修改成（`main.js`）：

```javascript
import './assets/css/main.css'

import { createApp } from 'vue'
import { createPinia } from 'pinia'
import Antd from 'ant-design-vue';
import 'ant-design-vue/dist/reset.css';
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'
import App from './App.vue'
import router from './router'
import axios from "axios";

axios.defaults.baseURL = 'http://localhost:8080'//设置全局的基础路径

axios.defaults.withCredentials=true;
//后端基础url 之后在请求时只用填写路径 Axios会自动以该url为基础添加路径

const app = createApp(App)
app.config.globalProperties.$axios = axios
app.use(createPinia())
app.use(router).use(ElementPlus).use(Antd);

app.mount('#app')


```
