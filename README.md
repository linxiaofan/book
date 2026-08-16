# 📚 书香阁图书馆 · HTML 协作练习项目

> 一个用于练习 **Git 分支管理** 与 **合并冲突解决** 的纯 HTML 工程。

本项目模拟一个小型图书馆网站，包含三个功能模块。每位同学负责一个模块，在独立分支上开发，最后合并回主分支。在这个过程中，你会经历**干净合并**与**真实冲突**两种情况。

---

## 📁 目录结构

```
book/
├─ index.html        ← 书籍页面（= 首页）【模块 A】
├─ about.html        ← 图书馆介绍【模块 B】
├─ librarian.html    ← 图书管理员介绍【模块 C】
├─ css/
│  └─ style.css      ← 共享样式表【全员编辑 · 冲突点 ①】
├─ README.md         ← 项目说明 + 贡献者名单【全员编辑 · 冲突点 ②】
├─ .gitignore
└─ docs/plans/       ← 设计与实施文档
```

## 🧩 三大功能模块

| 模块 | 文件 | 负责人  | 合并预期 |
|------|------|------|----------|
| 🅰 书籍（首页） | `index.html` | 白木白  | 多半干净合并 |
| 🅱 图书馆介绍 | `about.html` | 同学 林 | 干净合并 |
| 🅲 管理员介绍 | `librarian.html` | 同学 C | 干净合并 |
| 🎨 共享样式 | `css/style.css` | 全员   | **会产生冲突** |
| 📝 项目说明 | `README.md` | 全员   | **会产生冲突** |

> **关键**：三个 `.html` 是各自独立的，合并时通常不冲突；而 `style.css` 和 `README.md` 大家都会改，是刻意设计的冲突练习点。

---

## 🚀 快速开始

```bash
# 1. 克隆仓库
git clone <仓库地址>
cd book

# 2. 用浏览器打开首页（双击 index.html 即可）
```

无需安装任何依赖，纯 HTML + CSS，浏览器直接打开。

---

## 🌳 分支管理练习

### 第一步：创建你的分支

每位同学从 `main` 创建自己的功能分支：

```bash
# 同学 A（书籍页）
git checkout -b feature/books

# 同学 B（图书馆介绍）
git checkout -b feature/about

# 同学 C（管理员介绍）
git checkout -b feature/librarian
```

### 第二步：在你的分支上开发

修改你负责的模块文件，例如为书籍页新增一本书、为介绍页补充一条规则、为管理员页新增一位成员。

### 第三步：提交并推送

```bash
git add <你修改的文件>
git commit -m "feat: 描述你做了什么"
git push origin <你的分支名>
```

### 第四步：合并回主分支

方式一：直接合并（本地）
```bash
git checkout main
git pull origin main
git merge feature/books      # 合并你的分支
```

方式二：发起 Pull Request（推荐，更贴近真实工作流）
- 在代码托管平台（GitHub / Gitee）上从你的分支向 `main` 发起 PR，由负责人审核合并。

---

## ⚔️ 冲突演练场景

下面两个场景会**必然产生冲突**，请亲手体验并解决。

### 场景 ①：在 `css/style.css` 上制造冲突

每个人都在自己模块的区块末尾追加一段新样式，且都基于同一个旧版本提交。

```bash
# 1. 确保在 main 最新状态
git checkout main && git pull origin main

# 2. 切到你的分支并合并最新 main（此时可能已经冲突）
git checkout feature/books
git merge main
```

如果看到 `CONFLICT (content): Merge conflict in css/style.css`，说明冲突来了！打开文件会看到：

```css
<<<<<<< HEAD
/* 你分支里新加的样式 */
.book-card .badge { ... }
=======
/* main 上别人新加的样式 */
.book-card .ribbon { ... }
>>>>>>> main
```

**解决步骤：**
1. 用 `git status` 查看哪些文件冲突。
2. 打开冲突文件，找到 `<<<<<<<`、`=======`、`>>>>>>>` 标记。
3. 编辑文件，**保留你想要的内容**，删除三组冲突标记。
4. 标记已解决：`git add css/style.css`
5. 完成合并：`git commit`（Git 会自动填好合并信息，保存即可）

### 场景 ②：在 `README.md` 贡献者名单上制造冲突

每个人都下面的【贡献者名单】里加一行自己的信息，合并时相邻行必然冲突。解决方法同上。

> 💡 **冲突解决口诀**：`git status` 看战场 → 编辑文件留所需 → 删掉三组 `<<<<<<< ======= >>>>>>>` → `git add` 标记好 → `git commit` 完成它。

---

## 👥 贡献者名单

> 📌 练习用：请每位同学在自己的分支里把自己加到下面这个列表，合并时体验相邻行冲突。

- [ ] 模块 A（书籍页 `index.html`）— `<柏佳沂>`
- [ ] 模块 B（图书馆介绍 `about.html`）— `<讨厌你>`

- [ ] 模块 C（管理员介绍 `librarian.html`）— `<xiaolin和xiaobai>`


---

## 🛠️ 技术栈

- **HTML5**：语义化标签结构
- **CSS3**：Flexbox / Grid 布局、CSS 变量、媒体查询响应式
- 无 JavaScript、无构建工具、无后端，浏览器直接打开

---

## 📄 文档

- 设计文档：[`docs/plans/2026-08-13-library-html-project-design.md`](docs/plans/2026-08-13-library-html-project-design.md)
- 实施计划：[`docs/plans/2026-08-13-library-html-project.md`](docs/plans/2026-08-13-library-html-project.md)

---

© 2026 书香阁图书馆 · 本项目用于教学练习
