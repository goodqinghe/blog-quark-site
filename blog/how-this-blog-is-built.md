---
title: 这个博客是怎么搭起来的
updated: 2026-10-02
---

# 这个博客是怎么搭起来的

这个博客把 Markdown 文件转换成静态网页。源码在 `goodqinghe/blog-quark-src`，生成的文件交给 `goodqinghe/blog-quark-site`。下面按当前代码，走一遍从一篇原稿到页面的过程。

## 先看输入和输出

入口是 `scripts/build.mjs`。它从当前工作目录读取 `site.config.json`，配置里指定了：

```json
{
  "sourceDir": "content",
  "outputDir": "dist",
  "keepMarkdownInOutput": true
}
```

`content/` 是正式内容输入，`dist/` 是生成结果。构建会先清空 `dist/`，再递归读取原稿，所以应当改 `content/`，不能把 `dist/` 当作原稿目录。

以这篇文章为例：

```text
content/blog/how-this-blog-is-built.md
    ↓ npm run build
dist/blog/how-this-blog-is-built.html
```

目录关系会保留。图片、PDF 等非 Markdown 文件直接复制到对应位置；当前配置还会复制 Markdown 原稿，因此输出里同时有 `.md` 和 `.html`。

## 从 Markdown 读出什么

`readPage()` 先找文件开头的 Frontmatter，再提取标题和日期。例如：

```yaml
---
title: 这个博客是怎么搭起来的
updated: 2026-10-02
---
```

这里不是完整的 YAML 解析器：代码逐行读取简单的 `键: 值`，并去掉值两边的引号。标题优先取 `title`，其次取正文的第一个 H1，最后才用文件名。`updated` 用于首页“最近更新”的排序。

构建器没有根据 `draft: true` 跳过文章。放进 `content/` 的 Markdown 都会参与构建，包括没有进入首页推荐列表的文件。待审稿应留在正式输入目录之外，不能靠一个 Frontmatter 标记控制发布。

## 正文怎样变成 HTML

渲染器在 `scripts/markdown.mjs`，由 `markdown-it` 处理正文。生成链接时，代码会把以 `.md` 结尾的地址换成 `.html`，查询参数和片段标识可以保留。例如：

```text
../tech/linux-signal-learning.md
    ↓
../tech/linux-signal-learning.html
```

代码块分两种处理：

- 常用语言交给 Shiki，在构建时生成带高亮的 HTML。页面打开后不需要再运行高亮器。
- Mermaid 围栏先变成 `<pre class="mermaid">`。构建把 Mermaid 运行文件复制到 `dist/_site/mermaid/`；含有图表的页面再加载本站脚本，在浏览器里绘制 SVG。

没有匹配到高亮语言的代码块仍会输出，代码里的 HTML 特殊字符会被转义。普通代码块和 Mermaid 图表的渲染时机不同，检查问题时可以据此区分：前者看构建输出，后者还要看浏览器是否成功加载和执行脚本。

## 导航和页面外壳从哪里来

`site.config.json` 的 `navigation` 配置分类名、说明和文章顺序。`sectionDefinitions()` 把收集到的页面分到分类中，按配置排序；没有配置顺序的文章再按标题排序。

文章页由 `pageHtml()` 包上外壳，包括顶部入口、内容导航、面包屑和正文。桌面显示左侧竖栏，窄屏把同一份分类导航放进正文上方的折叠区；当前分类和当前文章仍有标识。这个切换主要由 CSS 和原生 `<details>` 完成。

首页使用 `homeHtml()` 单独生成，保留简介、“开始这里”、分类入口和“最近更新”，不放文章页的左侧竖栏。两种页面共用顶部入口、字体和颜色；首页负责选择入口，文章页负责阅读和切换同类文章。

样式也在 `build.mjs` 中生成。CSS 内容的 SHA-256 前 12 位进入文件名，页面引用类似 `_site/style.<hash>.css` 的地址；样式变化后，会生成新的文件名。`site-map.json` 则只记录标题、原稿路径和页面地址，不包含正文，也不是全文搜索索引。

## 本地如何检查

在源码仓库根目录安装依赖并构建：

```bash
npm ci
npm run build
```

要看实际页面，运行：

```bash
npm run dev -- --port 4173
```

`scripts/serve.mjs` 会先执行一次构建，再把 `dist/` 作为静态目录提供出来，默认只监听 `127.0.0.1`。浏览器访问 `http://127.0.0.1:4173`。它没有文件监听功能；改完原稿或模板，需要停止并重新启动预览。

## 推送之后发生什么

`.github/workflows/publish.yml` 配置了两个触发入口：推送源码仓库的 `main`，或手动执行工作流。工作流使用 Node.js 20 运行 `npm ci` 和 `npm run build`，然后将 `dist/` 同步到成品仓库 `goodqinghe/blog-quark-site` 的 `main`。

```text
blog-quark-src / main
    ↓ GitHub Actions 构建
dist/ 中的静态文件
    ↓ 同步
blog-quark-site / main
    ↓ Cloudflare Pages（按仓库 README 的部署配置）
浏览器访问页面
```

Cloudflare Pages 监听成品仓库的说明来自仓库 README，实际绑定和部署状态要到托管平台核对。源码中能直接确认的是构建与仓库间同步；这两步成功，也不能单独证明线上已经更新。

目前推送普通审稿分支不会命中这个 `main` 推送触发器。但合并到 `main` 会重新构建整个 `content/`，因此审一篇文章时，也要确认正式输入里没有其他不该发布的内容。

## 对照源码时看这几个文件

| 文件 | 负责的部分 |
| --- | --- |
| `site.config.json` | 输入输出目录、分类、排序和首页入口 |
| `scripts/build.mjs` | 收集原稿、生成页面和导航、复制资源、输出样式与清单 |
| `scripts/markdown.mjs` | Markdown、代码高亮和链接转换 |
| `scripts/serve.mjs` | 构建一次并提供本地静态预览 |
| `.github/workflows/publish.yml` | 在 `main` 更新后构建并同步成品仓库 |

页面显示不对，先对照原稿、渲染器和页面模板；本地结果正确而线上没变，再查构建工作流、成品仓库和托管部署。它们是连续的几步，也需要分别验证。
