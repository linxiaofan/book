# 图书馆 HTML 项目 Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** 搭建一个用于「分支管理 & 合并冲突」练习的图书馆纯 HTML 工程（3 模块页 + 2 共享冲突文件）。

**Architecture:** 三个独立模块页（`index.html`/`about.html`/`librarian.html`）+ 两个全员共享文件（`css/style.css`、`README.md`）。模块页之间干净合并；共享文件制造冲突。纯 HTML + CSS，无 JS、无构建、无后端。

**Tech Stack:** HTML5、CSS3（Flexbox/Grid、CSS 变量、媒体查询）。浏览器直接打开即用。

> **测试说明（纯静态站点）：** 无单元测试。每个任务的「验证」= 用浏览器打开页面检查渲染、点导航链接检查跳转、检查 HTML 标签闭合良好。

---

### Task 1: 项目脚手架 — `.gitignore`

**Files:**
- Create: `.gitignore`

**Step 1: 创建 `.gitignore`**

```
# 操作系统
.DS_Store
Thumbs.db
desktop.ini

# 编辑器
.vscode/
.idea/
*.swp
*.swo

# 其他
*.log
```

**Step 2: 验证**
- 确认文件存在且内容正确。

**Step 3: Commit**
```bash
git add .gitignore
git commit -m "chore: 添加 .gitignore"
```

---

### Task 2: 共享样式表 — `css/style.css`（冲突点 ①）

**Files:**
- Create: `css/style.css`

**Step 1: 写入共享样式（含分块注释）**

必须包含的分块（顺序固定，便于后续同学在自己区块追加）：
- `/* ===== 基础重置 / CSS 变量（共享，勿乱改）===== */`：`*{box-sizing}` reset、`:root` 主题变量（主色 `--primary`、次色 `--secondary`、背景、文字、卡片背景、圆角、阴影）、`body` 基础排版与字体。
- `/* ===== 通用：导航栏（共享）===== */`：`.site-header`、`.logo`、`.nav`、`.nav a`、`.nav a:hover`、`.nav a.active`（当前页高亮）、响应式导航。
- `/* ===== 通用：页脚（共享）===== */`：`.site-footer` 样式。
- `/* ===== 通用：布局容器 / 标题 ===== */`：`.container`（max-width 1100px 居中）、`.page-title`、`.section`。
- `/* ===== 模块A：书籍卡片 ===== */`：`.book-grid`（grid 自适应）、`.book-card`、`.book-cover`、`.book-title`、`.book-meta`、`.tag`（分类）、`.status`、`.status.available`（绿）/`.status.borrowed`（红）。
- `/* ===== 模块B：介绍页 ===== */`：`.info-grid`、`.info-card`、`.rules-list`、`.floor-list`、`.service-card`。
- `/* ===== 模块C：管理员页 ===== */`：`.team-grid`、`.member-card`、`.member-avatar`、`.member-name`、`.member-role`、`.member-detail`。
- `/* ===== 响应式 ===== */`：媒体查询（≤768px 单列）。

要求：使用 CSS 变量做主题色；卡片有圆角与浅阴影；导航在移动端可堆叠。

**Step 2: 验证**
- 文件无语法错误（可在 Task 3 渲染时一并检查）。

**Step 3: Commit**
```bash
git add css/style.css
git commit -m "feat: 添加共享样式表 style.css（含模块分块）"
```

---

### Task 3: 模块 A — `index.html`（书籍页面 / 首页）

**Files:**
- Create: `index.html`

**Step 1: 写入页面**

结构（全部用语义化标签）：
- `<head>`：`<meta charset>`、`<meta viewport>`、`<title>XX图书馆 - 书籍</title>`、`<link rel="stylesheet" href="css/style.css">`。
- `<header class="site-header">`：logo（XX图书馆）+ `<nav class="nav">` 三个链接，**首页设为 `class="active"`**（`index.html` / `about.html` / `librarian.html`）。
- 欢迎横幅 `<section class="hero">`：欢迎语 + 简介。
- `<section class="section"><h2>馆藏书籍</h2>`：`.book-grid` 包含约 8 张 `.book-card`。每张含：`.book-cover`（占位色块或 emoji）、`.book-title` 书名、`.book-meta`（作者·分类）、简介 `<p>`、馆藏地、`.status.available`/`.status.borrowed`。
  - 书目：《活着》余华（小说/可借）、《三体》刘慈欣（科幻/已借）、《百年孤独》马尔克斯（小说/可借）、《围城》钱钟书（小说/可借）、《小王子》圣埃克苏佩里（童话/可借）、《人类简史》赫拉利（历史/已借）、《解忧杂货店》东野圭吾（小说/可借）、《红楼梦》曹雪芹（古典/可借）。
- `<footer class="site-footer">`：版权、地址、联系方式。

**Step 2: 验证**
- 浏览器打开 `index.html`：导航三个链接均可跳转；书籍卡片网格渲染正常；借阅状态颜色区分；移动端缩放布局正常。

**Step 3: Commit**
```bash
git add index.html
git commit -m "feat: 添加书籍页面/首页 index.html"
```

---

### Task 4: 模块 B — `about.html`（图书馆介绍）

**Files:**
- Create: `about.html`

**Step 1: 写入页面**

结构：
- 同 Task 3 的 `<head>`，`<title>XX图书馆 - 馆况介绍</title>`。
- 同样的 `<header>` 导航，**「图书馆介绍」设为 `class="active"`**。
- `<section>` 图书馆简介：名称、成立年份、宗旨（2–3 段中文）。
- `<section class="section">` 基本信息 `.info-grid`：开放时间（周一至周日 8:30–22:00）、地址、联系电话、邮箱。
- `<section>` 馆藏规模与楼层分布 `.floor-list`：如 1F 借阅处、2F 文学社科、3F 科技外文、4F 自习与电子资源。
- `<section>` 借阅规则 `.rules-list`：可借数量、借期、续借次数、逾期规则。
- `<section>` 馆内服务 `.info-grid`/`.service-card`：自习室、电子资源、读者活动。
- 同 Task 3 的 `<footer>`。

**Step 2: 验证**
- 浏览器打开：导航高亮正确；各信息块布局正常；返回首页/管理员页链接有效。

**Step 3: Commit**
```bash
git add about.html
git commit -m "feat: 添加图书馆介绍页 about.html"
```

---

### Task 5: 模块 C — `librarian.html`（图书管理员介绍）

**Files:**
- Create: `librarian.html`

**Step 1: 写入页面**

结构：
- 同 Task 3 的 `<head>`，`<title>XX图书馆 - 管理员团队</title>`。
- 同样的 `<header>` 导航，**「管理员介绍」设为 `class="active"`**。
- `<section>` 团队简介（1–2 段）。
- `.team-grid` 包含约 5 张 `.member-card`：馆长、文学区管理员、科技区管理员、儿童区管理员、借阅处前台。每张含：`.member-avatar`（占位/emoji）、`.member-name` 姓名、`.member-role` 职位、`.member-detail`（负责区域、办公时间、联系方式）、一句话简介。
- 同 Task 3 的 `<footer>`。

**Step 2: 验证**
- 浏览器打开：导航高亮正确；团队卡片网格正常；链接有效。

**Step 3: Commit**
```bash
git add librarian.html
git commit -m "feat: 添加管理员介绍页 librarian.html"
```

---

### Task 6: `README.md`（分支练习指南 + 贡献者名单 = 冲突点 ②）

**Files:**
- Create: `README.md`

**Step 1: 写入 README**

内容章节：
- `# XX图书馆 · HTML 协作练习项目`：一句话简介 + 用途（练习分支管理与合并冲突）。
- `## 目录结构`：贴出项目树。
- `## 三大功能模块`：A 书籍（index.html）/ B 介绍（about.html）/ C 管理员（librarian.html），以及两个共享文件。
- `## 🚀 快速开始`：克隆、用浏览器打开 `index.html`。
- `## 🌳 分支管理练习`：
  - 建分支命令：`git checkout -b feature/books` / `feature/about` / `feature/librarian`。
  - 各分支职责表。
  - 提交推送：`git add` / `git commit -m` / `git push origin <分支>`。
  - 合并：`git checkout main` → `git merge <分支>`（或 PR）。
- `## ⚔️ 冲突演练场景`：
  - 场景①：三人都往 `css/style.css` 自己区块末尾追加新样式 → 末尾冲突。解决步骤：`git status` → 打开文件看 `<<<<<<<` / `=======` / `>>>>>>>` → 保留双方 → `git add` → `git commit`。
  - 场景②：三人都在 `README.md` 的「贡献者名单」同一列表里加自己一行 → 相邻行冲突。同法解决。
  - 一段「冲突解决口诀」。
- `## 👥 贡献者名单`：一个 Markdown 列表，留 3 行占位（`- [ ] 模块A - <姓名>` 等），供同学填写（冲突点）。
- `## 技术栈`：HTML + CSS，无 JS。

**Step 2: 验证**
- Markdown 预览正常；命令示例正确；贡献者名单为可填写列表。

**Step 3: Commit**
```bash
git add README.md
git commit -m "docs: 添加 README（分支练习指南 + 贡献者名单）"
```

---

## 完成标准（Definition of Done）

- [ ] 浏览器打开 `index.html`，三个导航链接可在 3 个页面间正确跳转。
- [ ] 每个页面当前导航项高亮（`active`）。
- [ ] 书籍/信息/管理员卡片在桌面与移动端布局正常。
- [ ] `style.css` 含清晰分块注释，便于练习时定位。
- [ ] `README.md` 含可执行的分支与冲突练习步骤。
- [ ] `.gitignore` 存在。
- [ ] 所有文件已分任务提交，`git log` 清晰。
