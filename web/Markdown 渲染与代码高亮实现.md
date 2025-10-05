#  **Markdown 渲染与代码高亮实现**

这段代码展示了如何使用 `markdown-it` 库在 Vue 应用中渲染 Markdown 内容并实现代码高亮。

## 基本 Markdown 渲染

1. **安装与导入**

   

   ```
   import MarkdownIt from 'markdown-it';
   import 'github-markdown-css/github-markdown.css'
   ```

2. **创建 MarkdownIt 实例**

   

   ```
   const md = new MarkdownIt({
     html: true,      // 允许 HTML 标签
     linkify: true,   // 自动转换链接
     typographer: true // 优化排版
   });
   ```

3. **在 Vue 模板中使用**

   

   ```
   <div 
     v-html="md.render(content)" 
     class="markdown-body"  <!-- 关键类名 -->
   />
   ```

## 代码高亮实现

1. **安装 highlight.js**

   

   ```
   npm install highlight.js
   ```

2. **导入样式和库**

   

   ```
   import 'highlight.js/styles/github.css' // 选择你喜欢的主题
   import hljs from 'highlight.js'
   ```

3. **配置 MarkdownIt 支持代码高亮**

   

   ```
   const md = new MarkdownIt({
     html: true,
     linkify: true,
     typographer: true,
     highlight: function (str, lang) {
       if (lang && hljs.getLanguage(lang)) {
         try {
           return hljs.highlight(str, { language: lang }).value;
         } catch (__) {}
       }
       return '';
     }
   })
   ```

## 关键点说明

1. **`markdown-body` 类**：应用 GitHub 风格的 Markdown 样式
2. **`v-html` 指令**：渲染 Markdown 转换后的 HTML 内容
3. **代码高亮**：
   - 根据代码块指定的语言进行高亮
   - 如果语言不支持或高亮失败，返回空字符串
   - 使用 `github.css` 主题，也可以选择其他主题