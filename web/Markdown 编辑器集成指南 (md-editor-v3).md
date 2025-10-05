# **Markdown 编辑器集成指南 (md-editor-v3)**

## 🌟 基本介绍

| 项目     | 说明                                                         |
| :------- | :----------------------------------------------------------- |
| 组件名称 | md-editor-v3                                                 |
| 框架支持 | Vue 3                                                        |
| 功能     | Markdown 编辑与预览                                          |
| GitHub   | [imzbf/md-editor-v3](https://github.com/imzbf/md-editor-v3)  |
| 语法文档 | [官方文档](https://imzbf.github.io/md-editor-v3/zh-CN/syntax) |

## 📦 安装方法

```
npm install md-editor-v3
```

## 🎨 组件模式对比

### ✍🏻 编辑器模式 (MdEditor)

| 特性     | 说明                   |
| :------- | :--------------------- |
| 双向绑定 | 支持 v-model           |
| 工具栏   | 内置完整Markdown工具栏 |
| 实时预览 | 支持分屏/全屏编辑      |
| 主题支持 | 亮色/暗色主题          |

```
<template>
  <MdEditor v-model="text" />
</template>

<script setup>
import { ref } from 'vue';
import { MdEditor } from 'md-editor-v3';
import 'md-editor-v3/lib/style.css';

const text = ref('# Hello Editor');
</script>
```

### 📖 仅预览模式 (MdPreview + MdCatalog)

| 特性     | 说明                      |
| :------- | :------------------------ |
| 只读模式 | 纯展示不可编辑            |
| 目录生成 | 配合MdCatalog自动生成目录 |
| 滚动同步 | 支持目录导航定位          |

```
<template>
  <MdPreview :editorId="id" :modelValue="text" />
  <MdCatalog :editorId="id" :scrollElement="scrollElement" />
</template>

<script setup>
import { ref } from 'vue';
import { MdPreview, MdCatalog } from 'md-editor-v3';
import 'md-editor-v3/lib/preview.css';

const id = 'preview-only';
const text = ref('# Hello Editor');
const scrollElement = document.documentElement;
</script>
```

## 🛠️ 常用配置项

| 属性               | 类型            | 说明                   | 适用组件  |
| :----------------- | :-------------- | :--------------------- | :-------- |
| v-model/modelValue | String          | 绑定Markdown内容       | 两者      |
| editorId           | String          | 组件唯一标识(必须唯一) | 两者      |
| theme              | 'light'\|'dark' | 主题设置               | 两者      |
| scrollElement      | HTMLElement     | 滚动容器(目录定位用)   | MdCatalog |
| noMermaid          | Boolean         | 是否禁用Mermaid图表    | MdPreview |
| noKatex            | Boolean         | 是否禁用Katex数学公式  | MdPreview |

## 💡 使用建议

1. **编辑器场景**：需要用户输入时使用`MdEditor`
2. **文档展示场景**：使用`MdPreview`+`MdCatalog`组合
3. **性能优化**：长文档建议启用懒加载
4. **样式定制**：通过CSS变量自定义主题颜色

## 🌈 扩展功能

```
- 支持GFM语法(GitHub Flavored Markdown)
- 内置代码高亮(Prism.js)
- 支持流程图、时序图(Mermaid)
- 数学公式渲染(Katex)
- 表情符号自动补全
- 图片上传集成
```