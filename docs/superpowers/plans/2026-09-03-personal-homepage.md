# 个人主页（轻量 Jekyll 单页）实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 按 `docs/superpowers/specs/2026-09-03-personal-homepage-design.md` 搭建一页学术风格个人主页（介绍块 = 页头：姓名居左、简介 | 照片+联系方式两栏；其下 News、Publications 同宽竖排），内容与样式分离，可一键发布到 GitHub Pages。

**Architecture:** 无插件的纯 Jekyll：`_layouts/home.html` 用 Liquid 渲染 `_data/news.yml` 与 `_data/publications.yml` 两个数据列表；`index.html` 只写简介与联系方式；一个自写 `assets/style.css` 控制布局。构建产物为静态 HTML，GitHub Pages 原生 Jekyll 构建直接兼容。

**Tech Stack:** Jekyll 4.x（已在 Gemfile/Gemfile.lock 中）、Liquid、原生 CSS（flexbox）。无 JS、无第三方主题、无 Jekyll 插件。

## Global Constraints

- 单页站点；不做 dark mode、博客、访问统计、多页面（spec 明确排除）。
- 不使用任何 Jekyll 插件或外部主题（保证 GitHub Pages 官方构建兼容）。
- 版心 `max-width: 750px` 居中；配色仅黑白灰（正文 `#222`，分隔线 `#ddd`，背景 `#fff`）。
- 窄屏断点 640px：照片+联系方式整栏折到简介上方（名片式），姓名 h1 保持最顶。
- 初版内容为占位示例（示例姓名/简介/2026–2025 示例论文、三条示例 news）；真实文案由用户在验收后替换。
- News 按日期倒序、Publications 按年份倒序分组，由模板排序保证，与 YAML 书写顺序无关。

## 文件结构

| 文件 | 操作 | 职责 |
|---|---|---|
| `.gitignore` | 新建 | 排除 `_site/`、`.jekyll-cache/`、`vendor/` |
| `Gemfile` / `Gemfile.lock` | 提交现有未跟踪文件 | Jekyll 依赖 |
| `_config.yml` | 新建 | 站点标题、作者名、语言、排除项 |
| `_data/news.yml` | 新建 | 新闻条目（date, text） |
| `_data/publications.yml` | 新建 | 论文条目（year, authors, title, venue, links?） |
| `assets/photo-placeholder.svg` | 新建 | 照片占位图 |
| `_layouts/home.html` | 新建 | 页骨架 + News/Publications Liquid 渲染 |
| `index.html` | 新建 | front matter + 两栏介绍块（简介、照片、联系方式） |
| `assets/style.css` | 新建 | 全部样式（约 120 行） |

任务切分：**Task 1** 交付"内容齐全但无样式"的可构建站点（结构 + 数据 + 排序渲染，全部用 `jekyll build` + `grep` 断言验证）；**Task 2** 交付样式与响应式（构建断言 + 浏览器目检）。两个任务各以一次提交结束。

---

### Task 1: 站点骨架、数据与排序渲染

**Files:**
- Create: `.gitignore`, `_config.yml`, `_data/news.yml`, `_data/publications.yml`, `assets/photo-placeholder.svg`, `_layouts/home.html`, `index.html`
- Commit（不修改）: `Gemfile`, `Gemfile.lock`

**Interfaces:**
- Consumes: 无（首个任务）
- Produces:
  - `_data/news.yml` 条目字段：`date`（YAML 日期 `YYYY-MM-DD`）、`text`（字符串）
  - `_data/publications.yml` 条目字段：`year`（整数）、`authors`、`title`、`venue`（字符串）、`links`（可选，子字段 `pdf`、`code`，值为 URL 字符串）
  - `_config.yml` 键：`title`、`author_name`、`lang`、`exclude` —— Task 2 的 CSS 依赖如下类名结构：`.page > header.intro > h1.name + .intro-cols > (.bio + aside.portrait > (img + .contact))`，以及 `.news`、`.publications`、`.pub-year`、`.pub-list`、`<time>`

- [ ] **Step 1: 写 `.gitignore`**

```
_site/
.jekyll-cache/
vendor/
```

- [ ] **Step 2: 写 `_config.yml`**

```yaml
title: "San Zhang — Homepage"
author_name: "San Zhang (张三)"
lang: en

exclude:
  - docs
  - vendor
  - Gemfile
  - Gemfile.lock
```

- [ ] **Step 3: 写 `_data/news.yml`（故意打乱日期顺序，用于验证模板排序）**

```yaml
- date: 2026-06-01
  text: "Sample news item C — 第三条示例新闻。"
- date: 2026-08-15
  text: "Sample news item A — 第一条示例新闻（日期最新，应排最前）。"
- date: 2026-07-01
  text: "Sample news item B — 第二条示例新闻。"
```

- [ ] **Step 4: 写 `_data/publications.yml`（故意先写 2025 再写 2026；并含"无 links 字段"条目以验证可选分支）**

```yaml
- year: 2025
  authors: "San Zhang, Si Li, Wu Wang"
  title: "An Earlier Sample Paper"
  venue: "ICML 2025"

- year: 2026
  authors: "San Zhang, Si Li"
  title: "A Sample Paper About Things"
  venue: "NeurIPS 2026 (preprint)"
  links:
    pdf: "https://example.com/paper.pdf"
    code: "https://github.com/example/repo"

- year: 2025
  authors: "San Zhang"
  title: "Another Sample Paper with Links"
  venue: "ICLR 2025"
  links:
    pdf: "https://example.com/paper2.pdf"
```

- [ ] **Step 5: 写 `assets/photo-placeholder.svg`**

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="400" height="500" viewBox="0 0 400 500">
  <rect width="400" height="500" fill="#e8e8e8"/>
  <text x="200" y="258" font-family="sans-serif" font-size="24" fill="#888" text-anchor="middle">Photo</text>
</svg>
```

- [ ] **Step 6: 写 `_layouts/home.html`**

关键约定：News 用 `sort: "date" | reverse` 倒序；Publications 用 `group_by: "year" | sort: "name" | reverse` 得到倒序年份组（Jekyll 内置 Liquid 没有 `uniq` 过滤器，勿改用 map+uniq 方案；同组内条目顺序不作要求）。

```html
<!DOCTYPE html>
<html lang="{{ site.lang | default: 'en' }}">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>{{ page.title | default: site.title }}</title>
  <link rel="stylesheet" href="{{ '/assets/style.css' | relative_url }}">
</head>
<body>
  <main class="page">

    <header class="intro">
      <h1 class="name">{{ site.author_name }}</h1>
      {{ content }}
    </header>

    <section class="news">
      <h2>News</h2>
      <ul>
        {% assign news = site.data.news | sort: "date" | reverse %}
        {% for item in news %}
        <li><time>{{ item.date | date: "%Y-%m" }}</time> — {{ item.text }}</li>
        {% endfor %}
      </ul>
    </section>

    <section class="publications">
      <h2>Publications</h2>
      {% assign years = site.data.publications | map: "year" | uniq | sort | reverse %}
      {% for y in years %}
      <h3 class="pub-year">{{ y }}</h3>
      <ul class="pub-list">
        {% for pub in site.data.publications %}
          {% if pub.year == y %}
        <li>
          {{ pub.authors }}. <em>{{ pub.title }}</em>. {{ pub.venue }}
          {%- if pub.links -%}
            <span class="links">
              {%- if pub.links.pdf %}<a href="{{ pub.links.pdf }}">[PDF]</a>{% endif -%}
              {%- if pub.links.code %}<a href="{{ pub.links.code }}">[Code]</a>{% endif -%}
            </span>
          {%- endif -%}
        </li>
          {% endif %}
        {% endfor %}
      </ul>
      {% endfor %}
    </section>

  </main>
</body>
</html>
```

- [ ] **Step 7: 写 `index.html`（联系方式与简介的占位文案写在此处）**

```html
---
layout: home
title: "San Zhang — Homepage"
---
<div class="intro-cols">
  <div class="bio">
    <p>I am a master's student in Computer Science at a sample university. My research interests include machine learning and systems. 这是一段占位个人简介，验收通过后请替换为真实文案。</p>
    <p>Second placeholder paragraph — 第二段占位简介。</p>
  </div>
  <aside class="portrait">
    <img src="{{ '/assets/photo-placeholder.svg' | relative_url }}" alt="Portrait of San Zhang">
    <div class="contact">
      <p>Email: <a href="mailto:you@example.com">you@example.com</a></p>
      <p><a href="https://scholar.google.com/">Google Scholar</a></p>
      <p><a href="https://github.com/username">GitHub</a></p>
    </div>
  </aside>
</div>
```

- [ ] **Step 8: 构建验证（本任务的"测试"）**

```bash
bundle exec jekyll build
```
预期：输出以 `Done in` 结尾、无 `ERROR`/`Liquid error` 字样。然后逐条断言 `_site/index.html`：

```bash
# a) 基础内容存在
grep -q 'you@example.com' _site/index.html && grep -q 'NeurIPS 2026' _site/index.html && grep -q '<time>2026-08</time>' _site/index.html && echo BASE-OK || echo BASE-FAIL
# b) News 日期倒序（A 行号 < B < C）
l1=$(grep -n '2026-08' _site/index.html | head -1 | cut -d: -f1); l2=$(grep -n '2026-07' _site/index.html | head -1 | cut -d: -f1); l3=$(grep -n '2026-06' _site/index.html | head -1 | cut -d: -f1); [ "$l1" -lt "$l2" ] && [ "$l2" -lt "$l3" ] && echo NEWS-ORDER-OK || echo NEWS-ORDER-FAIL
# c) 年份组倒序：2026 出现在 2025 之前
y1=$(grep -n '<h3 class="pub-year">2026' _site/index.html | cut -d: -f1); y2=$(grep -n '<h3 class="pub-year">2025' _site/index.html | head -1 | cut -d: -f1); [ "$y1" -lt "$y2" ] && echo YEAR-ORDER-OK || echo YEAR-ORDER-FAIL
# d) 2025 组含 2 篇论文（grep -c 该组条目）；无 links 的条目不渲染 [Code]
grep -q 'An Earlier Sample Paper' _site/index.html && grep -q 'Another Sample Paper with Links' _site/index.html && [ "$(grep -c '\[Code\]' _site/index.html)" = "1" ] && echo PUBS-OK || echo PUBS-FAIL
```
预期：四条 `*-OK`。任何 FAIL → 修 `_layouts/home.html` 后重跑本步。

- [ ] **Step 9: Commit**

```bash
git add .gitignore Gemfile Gemfile.lock _config.yml _data assets/photo-placeholder.svg _layouts index.html
git commit -m "feat: scaffold single-page homepage with data-driven news and publications"
```

---

### Task 2: 样式与响应式布局

**Files:**
- Create: `assets/style.css`
- 不修改其他文件（Task 1 的类名结构即其接口）

**Interfaces:**
- Consumes: Task 1 Step 6/7 中定义的 DOM 结构与类名。
- Produces: 最终视觉；浏览器目检通过后站点可发布。

- [ ] **Step 1: 写 `assets/style.css`**

```css
* { box-sizing: border-box; }

html { -webkit-text-size-adjust: 100%; }

body {
  margin: 0;
  background: #fff;
  color: #222;
  font-family: Georgia, "Times New Roman", "Songti SC", "SimSun", serif;
  font-size: 16px;
  line-height: 1.65;
}

.page {
  max-width: 750px;
  margin: 0 auto;
  padding: 2.5rem 1.25rem 4rem;
}

a { color: #222; text-decoration: underline; }
a:hover { color: #000; }

h1.name {
  margin: 0 0 1.4rem;
  font-size: 1.75rem;
  font-weight: 600;
  text-align: left;
}

h2 {
  margin: 2.4rem 0 0.8rem;
  font-size: 1.25rem;
  font-weight: 600;
  padding-bottom: 0.3rem;
  border-bottom: 1px solid #ddd;
}

.intro-cols {
  display: flex;
  gap: 2rem;
  align-items: flex-start;
}

.bio { flex: 1 1 auto; min-width: 0; }
.bio p:first-child { margin-top: 0; }

.portrait { flex: 0 0 210px; }
.portrait img {
  display: block;
  width: 100%;
  height: auto;
  border-radius: 8px;
}

.contact {
  margin-top: 0.8rem;
  font-size: 0.875rem;
}
.contact p { margin: 0.25rem 0; }

.news ul { margin: 0; padding-left: 1.1rem; }
.news li { margin-bottom: 0.35rem; }
.news time { font-weight: 600; }

.pub-year {
  margin: 1.4rem 0 0.4rem;
  font-size: 1.05rem;
  font-weight: 600;
}

.pub-list { list-style: none; margin: 0; padding: 0; }
.pub-list li { margin-bottom: 0.85rem; }
.pub-list .links a { margin-left: 0.3rem; text-decoration: none; }
.pub-list .links a:hover { text-decoration: underline; }

@media (max-width: 640px) {
  .intro-cols { flex-direction: column-reverse; }
  .portrait { flex: none; width: 180px; }
}
```

注意：`column-reverse` 使照片+联系方式整体折到简介**上方**（名片式），h1 姓名仍在介绍块顶部——这正是 spec 的窄屏要求。

- [ ] **Step 2: 构建断言**

```bash
bundle exec jekyll build 2>&1 | grep -Eq 'ERROR|Liquid error' && echo BUILD-FAIL || echo BUILD-OK
grep -q 'assets/style.css' _site/index.html && echo CSS-LINK-OK || echo CSS-LINK-FAIL
```
预期：`BUILD-OK`、`CSS-LINK-OK`。

- [ ] **Step 3: 浏览器目检（桌面 + 窄屏）**

```bash
bundle exec jekyll serve
```
打开 `http://127.0.0.1:4000`，验证：
1. 桌面宽度：版心居中 ≤750px；介绍块两栏（左简介、右照片+联系方式）；姓名在最顶、居左；各板块左边缘对齐。
2. 缩窗口到 <640px：照片+联系方式整体移到简介上方，姓名仍在最顶；无横向滚动条。
3. News 三条按 2026-08 → 07 → 06 排列；Publications 2026 一组（1 篇，含 [PDF][Code]）、2025 一组（2 篇，第一篇无任何链接）。
4. 黑白灰配色，无彩色。

任何不符 → 改 `assets/style.css` 后刷新重验。

- [ ] **Step 4: Commit**

```bash
git add assets/style.css
git commit -m "feat: add minimal black-white-gray styles and responsive layout"
```

---

## 验收后用户自行操作（不在本计划范围内）

1. 替换真实内容：`_config.yml` 的姓名、`index.html` 的简介/邮箱/链接、`_data/*.yml` 的真实 news/publications；把真实照片放进 `assets/` 并把 `index.html` 里的 `photo-placeholder.svg` 换成它。
2. 发布：新建 GitHub 仓库（用户站点则命名 `<用户名>.github.io`）→ push → 仓库 Settings → Pages → Build source 选 "Deploy from a branch"（master/main + /(root)），Jekyll 构建自动完成。
