# GET2Obsidian
基于Obsidian Importer插件的HTML导入功能进行扩展开发的Get笔记导入工具

核心实现包括：
- 增强HTML解析能力，实现对特定模板结构的智能识别
- 添加元数据提取功能，支持YAML front matter生成
- 优化Markdown转换逻辑，确保内容格式的准确转换

![效果示例](https://github.com/user-attachments/assets/e401a811-6ae0-4a59-9101-02b42e163959)
  
## Get笔记的导出模板结构

Get笔记导出的数据包含一个主索引文件 `index.html` 和一个 `notes` 文件夹，其中存放各个单独的笔记HTML文件。


**1. 主索引文件 (`index.html`) 结构:**

该文件通常包含一个笔记列表，其主要结构特点如下：
- 包含一个表格，用于列出所有笔记。
- 表格中的每一行 (`<tr>`) 代表一个笔记条目，通常具有 `class="date-list-item"`。
- 每一行包含以下关键信息：
    - 创建日期/修改日期 (`<td>`)。
    - 笔记标题，通常是一个链接 (`<td><a href="notes/note_file_name.html" target="_blank">笔记标题</a></td>`)，指向 `notes` 文件夹中对应的HTML文件。
    - 可能包含数据属性，如 `data-date-str`, `data-date`, `data-tags` 用于筛选和排序。

**2. 单个笔记文件 (`notes/xxxx.html`) 结构:**

每个笔记文件都是一个独立的HTML页面，其通用结构如下：

- **HTML基本结构:**
    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>笔记标题</title>
        <link rel="stylesheet" href="files/style.css">
        <script src="files/script.js"></script>
    </head>
    <body>
        <!-- 笔记内容 -->
        <script>
            // 通常包含一个初始化脚本，如：
            // document.addEventListener('DOMContentLoaded', initDetailPage);
        </script>
    </body>
    </html>
    ```

- **笔记内容容器:**
    - 通常有一个主要的容器 `div`，例如 `<div class="note-container">`。
    - 在此容器内部，有另一个 `div` 包含实际的笔记内容，例如 `<div class="note">`。

- **笔记元数据和内容:**
    - **标题:** 使用 `<h1>笔记标题</h1>`。
    - **创建日期:** 使用 `<p>创建于：YYYY-MM-DD HH:MM:SS</p>`。
    - **标签:** 使用 `<p>标签： <span class="tag">标签1</span> <span class="tag">标签2</span> ... </p>`。
    - **分隔线:** 通常有一个 `<hr>` 标签在元数据之后。
    - **正文:**
        - 主要内容通常包裹在 `<p>` 标签内。
        - 支持常见的HTML元素，如：
            - 列表: `<ul>`, `<ol>`, `<li>`
            - 链接: `<a href="...">...</a>`
            - 加粗: `<strong>...</strong>` 或 `<b>...</b>`
            - 下划线: `<u>...</u>`
            - 图片: `<img>` (需要注意图片路径，可能是相对路径或data URI)
            - 其他块级和内联元素。

**关键CSS选择器参考:**
- 笔记标题: `.note > h1`
- 创建日期: `.note > p` (需要结合文本内容 "创建于：" 进行区分)
- 标签容器: `.note > p` (需要结合文本内容 "标签：" 或 `span.tag` 子元素进行区分)
- 标签: `.note > p > span.tag`
- 笔记正文容器: 通常是 `<hr>` 标签之后的所有兄弟元素，或者一个特定的 `div` (如示例中的 `<div>` 直接在 `<hr>` 之后)。

在解析时，需要注意：
- 标题 (`<title>`) 和 `<h1>` 中的文本通常是一致的。
- `files/style.css` 和 `files/script.js` 是公共资源，可能位于 `notes` 文件夹的同级或父级。
- 附件（如图片）的路径需要正确处理，转换为Obsidian能识别的格式。

## 实现思路

    保留核心功能：保留原有的HTML解析、附件处理和Markdown转换功能
    添加配置选项：允许用户通过CSS选择器定义模板结构
    提取元数据：使用选择器从HTML中提取标题、标签、正文和创建时间
    生成前置元数据：将提取的元数据转换为YAML前置元数据
    使用提取的标题：使用提取的标题作为文件名，而不是原始文件名
