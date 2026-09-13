# 单元测试（UT）初学者 5 天学习方案（Python 版）

> **面向对象**：零测试基础的开发初学者。
> **建议投入**：每天 2~4 小时（理论 1 小时 + 动手 1~3 小时）。
> **技术栈**：Python + **pytest**（Python 世界事实上的标准测试框架，语法极简：普通函数 + `assert` 就能写测试；会了 pytest，再学 unittest / JUnit / Jest 都是换个语法而已）。

## 开始之前：本方案怎么用

1. **每天一个 Day，按顺序学**。Day1 是全书的"地图"，只讲概念不写代码，请务必先建立整体认知再动手。
2. **所有例子都基于本仓库**（书香阁图书馆网站）。本项目目前是纯 HTML + CSS，没有可测的 Python 代码——所以从 Day2 开始，我们会在项目里新增一个 `ut/` 目录、写少量业务函数作为"被测对象"。这也是真实工作的常态：**先有代码，才有单测**。
3. 每天末尾有 **作业** 和 **检验清单**。检验清单全打勾再进入下一天，否则宁可放慢一天。
4. 本方案的命令均为 **Windows PowerShell** 写法，macOS/Linux 用户把激活命令换成 `source .venv/bin/activate` 即可。

## 学完你能做到什么

- 用自己的话讲清楚：什么是测试、什么是测试用例、什么是单元测试
- 为任意一个功能**独立设计出覆盖完整的用例集**（等价类、边界值、判定表、场景法）
- 用 pytest 编写、运行、调试单元测试
- 用 Mock 替身隔离外部依赖
- 看懂覆盖率报告，用数据补齐漏测的用例

## 学习路线总览（自顶向下）

| 天 | 主题 | 一句话目标 |
|----|------|-----------|
| Day 1 | 全景与用例设计 | 看懂"测试"这件事的全貌，学会设计完整用例（不写代码） |
| Day 2 | 第一个单元测试 | 会搭环境，会把用例表翻译成可运行的测试代码 |
| Day 3 | 测试替身（Mock） | 会隔离外部依赖，让"依赖网络/时间"的代码也能单测 |
| Day 4 | 覆盖率与 TDD | 会用覆盖率数据检验用例完整性，体验测试先行 |
| Day 5 | 综合实战与规范 | 独立完成一个模块的完整测试套件，养成可持续的习惯 |

---

# Day 1：自顶向下 —— 先看懂"测试"与"用例"的全貌

> **今天的目标**：不写一行代码。搞懂三个问题——测试分几层？什么是用例？如何把用例设计"完整"？
> **今天最重要**：1.3 和 1.4 两节。这是整个测试领域的地基，后面 4 天全是在给它们落地。

## 1.1 为什么需要测试（15 min）

想象这个场景：你花了两天把 `css/style.css` 的合并冲突解决完，页面看起来没问题就合并了。一周后有人发现"书籍状态标签的颜色被改坏了"——因为冲突解决时误删了一段样式，而当时没有任何检查手段能发现它。

测试解决的就是这个问题：

| 没有测试 | 有测试 |
|---------|--------|
| 每次改完代码，靠人肉把所有页面点一遍 | 跑一条命令，几秒钟检查完所有关键功能 |
| 改了 A 功能，不知道会不会弄坏 B | 回归测试（regression test）自动告诉你 |
| 不敢重构、不敢改别人的代码 | 有测试兜底，敢放心改 |

> **核心概念：回归（Regression）**——以前好的功能，被后来的改动弄坏了。测试最大的价值就是防回归。

## 1.2 测试分层全景：测试金字塔（30 min）

软件测试自顶向下分三层，画成金字塔（越往下越多、越快、越便宜）：

```
        ／  E2E  ＼          ← 端到端测试：模拟真实用户从头到尾操作
       ／ 集成测试 ＼         ← 多个模块拼在一起，测它们的协作
      ／ 单元测试    ＼        ← 单元测试（UT）：测最小单元（函数/方法）
     ‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾
```

| 层 | 测什么 | 谁写 | 数量 | 速度 | 失败时定位问题 |
|----|--------|------|------|------|--------------|
| 单元测试（UT） | 一个函数/方法的逻辑 | 开发自己 | 最多（成百上千） | 毫秒级 | 最准（直接指向某行代码） |
| 集成测试 | 模块之间的协作（如"函数 + 数据库"） | 开发 | 中等 | 秒级 | 较准 |
| E2E / UI 测试 | 整个系统像用户一样操作 | 测试/开发 | 最少（核心路径） | 最慢 | 要排查整条链路 |

**为什么金字塔底最大？** 因为 UT 跑得最快、写起来最便宜、失败时定位最准——能用 UT 覆盖的逻辑就不要堆到上层去测。本方案 5 天全部聚焦最底层：**单元测试**。（E2E 那一层见 `docs/auto/自动化测试学习方案.md`。）

## 1.3 核心概念：什么是测试用例（90 min，今天最重要的内容）

### 一句话定义

> **测试用例（Test Case）= 在什么条件下（前置），执行什么操作（输入/步骤），期望得到什么结果（预期）。**

三要素缺一不可：

1. **前置条件**：执行前系统必须处于的状态
2. **操作步骤 / 输入**：做什么、传什么数据
3. **预期结果**：怎样算对——必须是**可判定的**，不能是"看起来正常"

生活类比：体检报告上的每一项就是一条用例——"空腹抽血（前置）→ 化验血糖（操作）→ 正常范围 3.9~6.1 mmol/L（预期）"。没有预期值，化验单就毫无意义。

### 用例先用"话"写，再用"代码"写

初学者最常见的误解是"测试用例 = 测试代码"。不对——**用例首先是一张表**，代码只是它后来的载体。我们拿图书馆的"借书"功能来示范。

功能描述：*读者借书。若书在馆可借，且该读者已借数量未达上限（5 本），则借出成功；否则拒绝并说明原因。*

给它设计用例表：

| 编号 | 用例标题 | 前置条件 | 操作 | 预期结果 | 设计方法 |
|------|---------|---------|------|---------|---------|
| TC01 | 可借的书正常借出 | 书《活着》状态=可借；读者已借 0 本 | 借《活着》 | 成功，书状态变为已借出 | 正常场景 |
| TC02 | 已借出的书不能借 | 书《三体》状态=已借出 | 借《三体》 | 拒绝，提示"已被借出" | 无效等价类 |
| TC03 | 达到上限后不能借 | 书可借；读者已借 5 本 | 借任意可借的书 | 拒绝，提示"超出上限" | 无效等价类 |
| TC04 | 差一本到上限可以借 | 书可借；读者已借 4 本 | 借书 | 成功 | 边界值 |
| TC05 | 图书不存在 | 输入一本系统中没有的书 | 执行借书 | 报错"图书不存在" | 异常输入 |

注意 TC03 和 TC04 的区别：一个在边界外、一个在边界内——**这就是"完整"的雏形**。

### 执行一条用例

用例写好后就可以"执行"：按前置条件准备好环境 → 执行操作 → 把实际结果与预期对比 → 记录**通过 / 失败**。手工执行是人工做；单元测试是把这三步写成代码让机器做——**内容完全一样，载体不同**。

### 常见误区

- ❌ "我点了一圈，没发现问题"——没有预期结果的操作不叫用例
- ❌ "页面看起来正常"——预期必须可判定：什么可见？文本是什么？数量是多少？
- ❌ 只测正常流程——TC02、TC03 这类"拒绝"用例往往才是藏 bug 的地方

## 1.4 如何覆盖"完整"的用例（90 min，今天第二重要的内容）

"完整"不是漫无目的地多写，而是有方法的。入门必会四招，每一招都用"借书"演示：

### 方法一：等价类划分——把无限种输入归类

输入的可能性是无限的（已借 0 本、1 本、2 本……），不可能每种种一条。**把"程序处理方式相同"的输入归为一类，每类选一个代表**：

- `已借数量` 的有效等价类：**[0, 4]**（都能借成功）→ 代表：0 本或 4 本
- `已借数量` 的无效等价类：**[5, +∞)**（都拒绝）→ 代表：5 本
- `书状态` 的等价类：{可借} → 成功；{已借出} → 拒绝

归类之后，无限输入变成了有限几条用例。

### 方法二：边界值分析——bug 最爱藏在边界上

程序员的循环条件、比较符号经常在边界上出错（`>` 写成 `>=`、`<` 写成 `<=`）。所以**每个边界，都要取"边界值本身 + 两侧相邻值"来测**：

`已借数量` 的上限是 5，则测 **4（边界内）、5（边界上）、6（边界外）**：

| 输入 | 预期 | 为什么 |
|------|------|--------|
| 已借 4 本 | 成功 | 边界内最大成功值 |
| 已借 5 本 | 拒绝 | 正好压在边界上 |
| 已借 6 本 | 拒绝 | 边界外（与 5 同类，可选） |

同样思路：逾期天数 0 和 1 之间、罚款封顶金额的临界天数，都是边界。

### 方法三：判定表——多条件组合全覆盖

当结果由**多个条件共同决定**时，列出条件的所有组合：

| 规则 | 条件①书可借？ | 条件②未达上限？ | 结果 |
|------|-------------|---------------|------|
| 1 | 是 | 是 | 借出成功 |
| 2 | 是 | 否 | 拒绝：超出上限 |
| 3 | 否 | 是 | 拒绝：已被借出 |
| 4 | 否 | 否 | 拒绝（提示书不可借） |

2 个条件 × 各 2 种取值 = 4 条规则，**每条规则至少一条用例**，一个组合都漏不掉。

### 方法四：场景法——沿着流程走，正常流 + 每条异常流

把功能想成一条流程：`选书 → 校验书状态 → 校验借阅上限 → 借出`。

- **正常流**：全部校验通过，成功借出（TC01）
- **异常流**：在每个可能失败的节点分岔出去——书不可借（TC02）、超上限（TC03）、书不存在（TC05）

> 完整用例集 = 正常场景 + 每一种异常场景 + 每个边界。

### 完整性自检三问（写完用例后必问）

1. 代码里**每个判断的每个方向**（每个 if 的真/假两个分支），都有用例经过吗？
2. **每个数字的边界**（0、最大值、最大值+1），都被碰到了吗？
3. **每种非法输入**（None、不存在的数据、类型错误），都有用例吗？

三问都答"是"，用例集就基本完整了。明天你会看到：单测代码里的每个 `test_` 函数，就对应表里的一行。

## 1.5 那么，"单元测试"到底是什么（30 min）

- **单元（Unit）**：最小的可测单位，通常是**一个函数或一个方法**。
- **单元测试**：把 1.3 的用例表写成代码——每个 `test_` 函数是表中的一行，机器执行，自动判定通过/失败。

一个 UT 长什么样（先混个眼熟，明天亲手写）：

```python
def test_borrowed_book_cannot_borrow():
    """已借出的书不能借"""
    book = {"title": "三体", "status": "borrowed"}   # 前置条件
    result = can_borrow(book, 0)                      # 操作
    assert result is False                            # 预期结果（断言）
```

**好的单元测试满足 FIRST 原则**：

| 原则 | 含义 |
|------|------|
| **F**ast | 跑得快（毫秒级），才敢天天跑 |
| **I**ndependent | 用例之间互不依赖，单独跑任何一条都行 |
| **R**epeatable | 任何环境、任何时间跑，结果都一样 |
| **S**elf-validating | 自动判定通过/失败，不需要人看输出猜 |
| **T**imely | 与代码同步写（甚至先写测试），不是项目结尾补作业 |

## 今日作业（不写代码）

为"**还书**"功能手写用例表。功能描述：*读者归还一本书。书必须是本馆藏书且处于"已借出"状态；归还成功后书状态变为"可借"。若书不存在或本就未被借出，给出对应提示。*

要求：≥ 6 条用例，用上等价类、边界值（提示：想想"借出状态"的两种取值）、场景法（正常流 + 异常流），并标注每条用例的设计方法。

## 今日检验清单

- [ ] 我能用一句话说出测试用例的三要素
- [ ] 我能画出测试金字塔，说出三层各自的作用
- [ ] 我能为"还书"列出等价类和边界值
- [ ] 我知道"完整"的用例集要过"自检三问"
- [ ] 我能说出 FIRST 原则中至少 3 条

---

# Day 2：动手写第一个单元测试（pytest）

> **今天的目标**：搭好环境，写出第一批可运行的单元测试，体会"用例表 → 测试代码"的翻译过程。

## 2.1 环境准备（30 min）

1. 安装 Python（3.10+）：https://www.python.org/downloads/ ，安装时**勾选 "Add Python to PATH"**。验证：

```powershell
python --version    # 显示 3.10 及以上即成功
```

2. 在项目根目录 `C:\project\book` 下创建虚拟环境并安装 pytest：

```powershell
cd C:\project\book
python -m venv .venv                  # 创建虚拟环境（项目私有的 Python 环境）
.venv\Scripts\Activate.ps1            # 激活（macOS/Linux: source .venv/bin/activate）
pip install pytest pytest-cov
pytest --version                      # 验证安装成功
```

> **如果激活报错**（PowerShell 提示"禁止运行脚本"），执行一次：
> `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser`，然后重新打开终端再激活。
>
> 以后**每次打开新终端，先激活虚拟环境再干活**（提示符前面出现 `(.venv)` 就是激活了）。

3. 在 `.gitignore` 里加几行（这些不该进 git）：

```
.venv/
__pycache__/
.pytest_cache/
.coverage
htmlcov/
```

## 2.2 测试文件的骨架（30 min）

pytest 的约定非常简单：

- 测试文件命名为 `test_*.py` 或 `*_test.py`
- 测试函数命名为 `test_*`
- 断言直接用 Python 原生的 `assert` 语句

新建 `ut/test_demo.py`（在项目根目录下建 `ut/` 文件夹）感受结构：

```python
# ut/test_demo.py

def test_add():
    """1 + 1 等于 2"""
    assert 1 + 1 == 2

def test_will_fail_on_purpose():
    """这个例子故意失败，看看失败长什么样"""
    assert 1 + 1 == 3
```

在项目根目录运行：

```powershell
pytest ut               # 只跑 ut/ 目录下的测试
pytest ut -v            # -v 显示每条用例的名字（学习期建议加）
```

观察输出：`.` 表示通过、`F` 表示失败，失败信息会**精确显示表达式的实际值**（比如 `assert (1 + 1) == 3` 下面标注 `+2 where 1 + 1 = 2`）。**读懂失败输出和写出测试同样重要**。

> 注意：运行命令一律带目录参数（`pytest ut`），不要在根目录裸跑 `pytest`——本项目还有别的测试目录，裸跑会把它们一起收进来。

## 2.3 被测代码：借书函数（20 min）

新建 `ut/borrow.py`——这就是我们要测的"单元"：

```python
# ut/borrow.py
"""借阅规则：判断一本书当前能否被某位读者借走"""

MAX_BORROW = 5  # 每位读者最多同时借 5 本


def can_borrow(book, borrowed_count):
    """判断能否借书。

    book: 图书字典，形如 {"title": "活着", "status": "available"}
    borrowed_count: 该读者当前已借数量
    返回 True=可以借 / False=不能借；图书不存在时抛 ValueError
    """
    if book is None:
        raise ValueError("图书不存在")
    if book.get("status") != "available":
        return False  # 已借出 / 不可外借
    if borrowed_count >= MAX_BORROW:
        return False  # 超出可借上限
    return True
```

对照 Day1 的判定表看：这个函数的三个 `if` 正好对应"书不存在 / 书不可借 / 超上限"三条规则。

## 2.4 把 Day1 的用例表翻译成测试代码（60 min）

新建 `ut/test_borrow.py`。**表里每一行 = 一个 `test_` 函数**，每个函数内部遵循 **AAA 模式**：

```python
# ut/test_borrow.py
import pytest
from borrow import can_borrow, MAX_BORROW


def test_available_book_with_zero_borrowed_can_borrow():
    """TC01 可借的书、读者已借 0 本 → 可以借"""
    book = {"title": "活着", "status": "available"}   # Arrange 准备：构造输入
    result = can_borrow(book, 0)                       # Act 执行：调用被测函数
    assert result is True                              # Assert 断言：验证结果


def test_available_book_below_limit_can_borrow():
    """TC04 可借的书、读者已借 4 本（上限-1）→ 可以借"""
    book = {"title": "围城", "status": "available"}
    assert can_borrow(book, MAX_BORROW - 1) is True


def test_borrowed_book_cannot_borrow():
    """TC02 已借出的书 → 不能借"""
    book = {"title": "三体", "status": "borrowed"}
    assert can_borrow(book, 0) is False


def test_at_limit_cannot_borrow():
    """TC03 读者已借 5 本（达到上限）→ 不能借"""
    book = {"title": "小王子", "status": "available"}
    assert can_borrow(book, MAX_BORROW) is False


def test_missing_book_raises():
    """TC05 图书不存在 → 抛出异常"""
    with pytest.raises(ValueError, match="图书不存在"):
        can_borrow(None, 0)
```

跑 `pytest ut -v`，应当 **5 条全绿**（每个函数名对应用例表的一行——这就是 pytest 里"用例名讲业务"的方式：英文函数名 + 中文 docstring）。然后做两个实验：

1. **改坏实现**：把 `borrow.py` 里的 `>=` 改成 `>`，再跑——"已借 5 本"的用例变红。这就是测试在**捕获回归**。改回来。
2. **改错断言**：把某条断言的 `True` 改成 `False`，再跑——看失败信息如何精确告诉你期望与实际。改回来。

> 测试函数为什么要写在 `test_borrow.py` 里就能 `from borrow import ...`？因为 pytest 会自动把测试文件所在目录加入导入路径——同目录的 `borrow.py` 直接 import 即可。

## 2.5 常用断言写法速查（20 min）

pytest 的断言就是普通 `assert`，配合 `raises` 处理异常：

| 场景 | 写法 |
|------|------|
| 等于 | `assert result == 30` |
| 布尔（严格） | `assert result is True` / `assert result is False` |
| 包含 | `assert "小说" in tags` |
| 近似比较（浮点） | `assert result == pytest.approx(3.5)` |
| 抛异常 | `with pytest.raises(ValueError, match="图书不存在"):` |

## 今日作业

实现并测试逾期罚款函数 `ut/fine.py`。规则：

*逾期天数 ≤ 0 → 罚款 0 元；1~7 天 → 每天 0.5 元；超过 7 天 → 前 7 天按 0.5 元/天，之后每天 1 元，总额封顶 30 元。*

要求：
1. **先手写用例表再写代码**（Day1 的方法）：找出所有边界（0 和 1 之间、7 和 8 之间、封顶的临界天数是多少？自己算）
2. 用例 ≥ 7 条（函数名 `test_*`，docstring 写中文用例标题），全部通过 `pytest ut -v`
3. 涉及 0.5 元的断言记得用 `pytest.approx`

## 今日检验清单

- [ ] 我能独立完成 venv 创建、激活、安装 pytest、`pytest ut` 跑通
- [ ] 我能说出 pytest 的三条命名约定
- [ ] 我把用例表的每一行翻译成了一个 `test_` 函数，并遵循 AAA 模式
- [ ] 我故意改坏过实现和断言，并读懂了失败输出
- [ ] 作业 `calc_fine` 的用例表覆盖了所有边界

---

# Day 3：测试替身（Mock）—— 隔离依赖

> **今天的目标**：让"依赖外部世界"的代码（发邮件、读数据库、当前时间）也能被快速单测。

## 3.1 问题引入（20 min）

`can_borrow` 好测，因为它只吃参数、返回结果，不碰外部世界。但看这个需求：

> *到期前一天，给读者发一封提醒邮件。*

发邮件要连邮件服务器——测试时真的发一封？太慢、不稳定、还会骚扰别人。**单元测试的边界原则：只测当前单元的逻辑，它的依赖全部用"替身"顶替。** 电影用替身演员完成危险动作，测试用"替身对象"完成外部交互。

## 3.2 三种替身（40 min）

初学者先记住三种就够：

| 替身 | 干什么 | 例子 |
|------|--------|------|
| **Stub（桩）** | 返回假数据，让代码能跑下去 | 假的数据库，固定返回某本书 |
| **Spy（间谍）** | 正常放行，同时**记录**发生了什么调用 | 记录"send 被调了 1 次、参数是什么" |
| **Mock（模拟）** | 替身 + **验证**交互是否符合预期 | 断言"邮件函数被调用且参数正确" |

Python 标准库自带替身工具：`unittest.mock.MagicMock`——一个"什么都不做、但把一切调用记录下来"的万能替身，一个对象同时充当 Spy 和 Mock。

## 3.3 先学最朴素的隔离手法：依赖注入（30 min）

怎么把真邮件服务换成替身？最简单的方式：**把依赖作为参数传进来**。

- 生产环境：传入真的邮件服务
- 测试里：传入 `MagicMock()`

函数本身根本不关心 `mailer` 是真是假——只管调用它。这就是**依赖注入（Dependency Injection）**，没有黑魔法。

## 3.4 动手：给提醒功能写测试（60 min）

被测代码 `ut/reminder.py`：

```python
# ut/reminder.py
"""到期提醒：到期前 1 天发送一封提醒邮件"""


def send_due_reminder(book, mailer, days_until_due):
    """到期前 1 天通过 mailer 发提醒邮件。

    mailer 是"发邮件的能力"，由外部注入：
    生产环境传真邮件服务，测试里传 MagicMock 替身。
    """
    if days_until_due != 1:
        return False  # 还没到提醒时间
    mailer.send(f"【书香阁】《{book['title']}》将于明天到期，请按时归还。")
    return True
```

测试 `ut/test_reminder.py`：

```python
# ut/test_reminder.py
from unittest.mock import MagicMock
from reminder import send_due_reminder


def test_sends_reminder_one_day_before_due():
    """到期前 1 天 → 发送 1 封提醒邮件"""
    mailer = MagicMock()                       # 替身：记录一切调用

    result = send_due_reminder({"title": "活着"}, mailer, days_until_due=1)

    assert result is True                                # 返回值对不对
    mailer.send.assert_called_once_with(                 # 调了几次、参数是什么
        "【书香阁】《活着》将于明天到期，请按时归还。"
    )


def test_no_reminder_three_days_before_due():
    """到期前 3 天 → 不发送"""
    mailer = MagicMock()

    result = send_due_reminder({"title": "活着"}, mailer, days_until_due=3)

    assert result is False
    mailer.send.assert_not_called()
```

注意这里的用例设计：正反两条（到期前 1 天 / 不是 1 天），又是一次等价类划分。

Mock 的常用断言一览：

| 断言 | 含义 |
|------|------|
| `mock.assert_called_once_with(参数)` | 恰好调用 1 次且参数正确 |
| `mock.assert_not_called()` | 从未被调用 |
| `mock.call_count` | 调用次数（可 `assert mock.send.call_count == 2`） |
| `mock.return_value = 42` | 让替身调用后返回指定值（Stub 的用法） |

## 3.5 pytest 自带的替身工具：monkeypatch（知道即可）

pytest 还提供 `monkeypatch` 夹具（fixture），可以临时替换模块里的函数或属性，测试结束自动还原：

```python
def test_something(monkeypatch):
    monkeypatch.setattr(某模块, "获取当前时间", lambda: "2026-09-13")
```

测"当前时间""随机数"这类全局依赖时很好用。需要时查官方文档，今天不深入。

## 今日作业

实现并测试逾期通知 `ut/notify.py`：

*`notify_overdue(book, mailer, overdue_days)`：逾期天数 > 0 时，通过 mailer 发送逾期通知并返回 True；否则不发邮件返回 False。通知文案自拟，但必须包含书名。*

要求：至少 3 条用例（正常通知 / 不通知 / 断言文案包含书名——提示：`mailer.send.assert_called_once()` 后用 `mock.send.call_args` 取出参数再做 `assert "活着" in 文案`）。

## 今日检验清单

- [ ] 我能说出为什么发邮件、读数据库的代码不能直接单测
- [ ] 我能区分 Stub 和 Mock 的用途
- [ ] 我会用 `MagicMock` 做替身，并断言 `assert_called_once_with` / `assert_not_called`
- [ ] 我理解依赖注入：把依赖变成参数，测试时传替身

---

# Day 4：覆盖率 —— 用数据检验用例是否完整

> **今天的目标**：让机器告诉你"哪些代码从没被任何用例执行过"，并体验测试先行（TDD）。

## 4.1 什么是覆盖率（30 min）

覆盖率（Coverage）回答一个问题：**测试跑的时候，代码有多少被执行到了？**

| 指标 | 含义 |
|------|------|
| 行覆盖 | 多少行代码被跑过 |
| 分支覆盖 | 多少个 if 的真/假方向都被走到过（更严格，更接近 Day1 的"自检三问"） |
| 函数覆盖 | 多少个函数被调用过 |

Day2 已装好 `pytest-cov`，直接用命令开启（运行都在项目根目录）：

```powershell
pytest ut --cov=ut --cov-branch --cov-report=html
```

- `--cov=ut`：统计 `ut/` 目录下代码的覆盖率
- `--cov-branch`：开启**分支覆盖**（比行覆盖更严格，推荐默认开启）
- `--cov-report=html`：生成 HTML 报告

## 4.2 跑报告，看懂红绿（40 min）

做个实验：先**注释掉** `test_borrow.py` 里"图书不存在"那条用例（函数前加一行 `#` 不行，直接临时删掉或重命名成 `def x_test_missing_book_raises():`），然后跑上面命令。

终端会输出覆盖率表格；同时生成 `htmlcov/` 目录，浏览器打开 `htmlcov/index.html`，点进 `borrow.py` 可以看到**逐行标注的源码**：绿色=被测过，**灰色/红色=从没有被任何用例执行**。你会看到 `raise ValueError("图书不存在")` 那一行没被覆盖——删掉的那条用例是它唯一的"顾客"。

把用例加回来，重新跑，它变绿。**这就是覆盖率的价值：用例完整性第一次变成了看得见的数据。**

## 4.3 覆盖率的陷阱（30 min）

**100% 覆盖 ≠ 没有 bug。** 看这两条"绿色"的测试：

```python
# 坏例子 1：跑了代码，但没有任何断言 —— 永远绿，什么也没验证
def test_no_assertion_fake():
    can_borrow({"title": "活着", "status": "available"}, 0)

# 坏例子 2：断言了一个错误的预期 —— 覆盖有了，但验证的是错的
def test_wrong_expectation():
    assert can_borrow({"title": "三体", "status": "borrowed"}, 0) is True  # 明明该是 False
```

结论：

- 覆盖率高只保证"代码被执行过"，**不保证"执行结果被正确验证"**
- 覆盖率是"完整用例"的**必要条件而非充分条件**
- 正确姿势：**用例设计靠方法（Day1），完整性校验靠覆盖率（今天）**——两者配合

## 4.4 TDD 初体验：红 → 绿 → 重构（60 min）

TDD（Test-Driven Development，测试驱动开发）：**先写测试，再写实现**。

以判断逾期为例，三步走：

1. **红**：先写测试（此时 `loan.py` 还不存在）——

```python
# ut/test_loan.py
from loan import is_overdue


def test_before_due_date_not_overdue():
    """今天早于到期日 → 未逾期"""
    assert is_overdue("2026-09-13", "2026-09-12") is False


def test_on_due_date_not_overdue():
    """今天是到期日当天 → 未逾期"""
    assert is_overdue("2026-09-13", "2026-09-13") is False


def test_after_due_date_overdue():
    """今天晚于到期日 → 逾期"""
    assert is_overdue("2026-09-13", "2026-09-14") is True
```

   运行 → 报错找不到模块 `loan`，全红。
2. **绿**：写最简单的实现让它通过——

```python
# ut/loan.py
"""判断是否逾期（ISO 格式日期字符串 "2026-09-13" 可直接比较大小）"""


def is_overdue(due_date, today):
    return today > due_date
```

   运行 → 全绿。
3. **重构**：在测试保护下优化代码结构，随时跑测试确认没改坏。

体会一下顺序颠倒的好处：**用例先行 = 强迫你在写实现之前，先把边界想清楚**（"当天算不算逾期？"这个问题在写实现前就被用例逼着回答了）。

## 4.5 实战：把分支覆盖率打到 100%（40 min）

新函数 `ut/rules.py`：

```python
# ut/rules.py
"""不同读者类型的一次可借天数"""


def get_borrowable_days(reader_type):
    if reader_type == "student":
        return 30
    if reader_type == "teacher":
        return 60
    if reader_type == "guest":
        return 7
    raise ValueError("未知的读者类型：" + reader_type)
```

1. 只写一条用例（测 `student`），跑覆盖率 → 看 Branch 覆盖率多低
2. 打开 `htmlcov/` 报告，找到没被走过的分支
3. 按判定表思路补齐：`teacher`、`guest`、非法类型（用 `pytest.raises`）→ 再跑到分支 100%

**这个过程就是 Day1"自检三问"的机器化版本。**

## 今日作业

1. 给 Day2 的 `calc_fine` 跑覆盖率，把分支覆盖补到 100%
2. 检查全部测试里有没有"没有断言"的假测试
3. （选做）用 TDD 方式实现 `get_borrowable_days_text(reader_type)`：返回"学生可借 30 天"这类文案，非法类型报错

## 今日检验清单

- [ ] 我会跑 `pytest ut --cov=ut --cov-branch --cov-report=html` 并看懂表格和 htmlcov 报告
- [ ] 我能举出"覆盖率 100% 但测试没价值"的例子
- [ ] 我体验了 红→绿 的 TDD 循环，理解"用例先行"倒逼想清边界
- [ ] 我的 `calc_fine` 分支覆盖率 100%，且每条测试都有有效断言

---

# Day 5：综合实战 + 测试规范 + 下一步

> **今天的目标**：把前四天的技能合成一个完整、可维护的测试套件，并建立长期习惯。

## 5.1 综合实战：借阅规则模块的完整测试套件（90 min）

确认项目结构长这样：

```
book/
├─ index.html / about.html / librarian.html
├─ css/style.css
├─ ut/
│  ├─ borrow.py        ← 借书规则        (Day2)
│  ├─ test_borrow.py
│  ├─ reminder.py      ← 到期提醒        (Day3)
│  ├─ test_reminder.py
│  ├─ notify.py        ← 逾期通知        (Day3 作业)
│  ├─ test_notify.py
│  ├─ fine.py          ← 逾期罚款        (Day2 作业)
│  ├─ test_fine.py
│  ├─ loan.py          ← 是否逾期        (Day4)
│  ├─ test_loan.py
│  └─ rules.py         ← 读者类型规则    (Day4)
│     └─ test_rules.py
├─ .venv/              (gitignore)
├─ requirements.txt
└─ .gitignore
```

任务清单：

1. 固化依赖清单——项目根目录新建 `requirements.txt`：

```
pytest
pytest-cov
```

   以后新环境一条命令装齐：`pip install -r requirements.txt`

2. `pytest ut -v` —— 全部测试一次性通过
3. `pytest ut --cov=ut --cov-branch` —— 核心逻辑分支覆盖率达到 100%
4. **加一个参数化用例**（Day2 遗留的重复代码，现在解决）。`@pytest.mark.parametrize` 把"同一逻辑、多组数据"的用例压成一条：

```python
# 追加到 ut/test_borrow.py
@pytest.mark.parametrize(
    "borrowed_count, expected",
    [
        (0, True),                # 下边界
        (MAX_BORROW - 1, True),   # 上限 - 1
        (MAX_BORROW, False),      # 正好压在上限
        (MAX_BORROW + 1, False),  # 上限 + 1
    ],
)
def test_can_borrow_boundary(borrowed_count, expected):
    """可借的书 + 各边界已借数量"""
    book = {"title": "活着", "status": "available"}
    assert can_borrow(book, borrowed_count) is expected
```

5. **回归实验**：把 `borrow.py` 里 `borrowed_count >= MAX_BORROW` 改成 `>`，跑 `pytest ut`——参数化用例立刻揪出"上限+1 本还能借"的 bug。体会：**改坏一行代码，3 秒钟被抓住。**

## 5.2 测试代码规范 6 条（30 min）

1. **用例名讲业务，不讲代码**：docstring 写"已借出的书不能借"，不写"测试 status 为 borrowed 时返回 False"
2. **一条 test 只验证一件事**：发现一个 test 里堆了多个不相关断言，就拆开
3. **用例之间独立**：不依赖执行顺序，不共享可变状态（FIRST 的 I）
4. **不测实现细节**：验证"返回什么"，而不是"内部怎么算的"——否则重构必挂测试
5. **测试也是代码**：重复了就参数化、抽公共函数，保持整洁
6. **失败信息要能读懂**：三个月后测试红了，你要能只看用例名 + 断言就知道哪坏了

## 5.3 接入日常工作流（30 min）

`pytest ut` 已经是你的日常命令：**每改一次业务代码，就跑一次**。

更进一步可以交给 CI（持续集成）：把代码推到 GitHub 后，让服务器自动跑测试。在 `.github/workflows/test.yml` 写：

```yaml
name: unit-tests
on: [push]                # 每次推送自动触发
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install -r requirements.txt
      - run: pytest ut --cov=ut --cov-branch
```

效果：任何人 push 代码，GitHub 自动跑全部测试并在页面上标红/标绿——测试成了团队的质量门禁。（今天只需理解概念，具体可在有 GitHub 仓库后实践。）

## 5.4 五天成果自测（20 min）

不看资料回答，答不上来的回到对应 Day 复习：

1. 测试用例的三要素是什么？（Day 1.3）
2. 画出测试金字塔，说明为什么 UT 在最底层且数量最多？（Day 1.2）
3. 等价类划分和边界值分析分别解决什么问题？对"已借数量上限 5"各给出哪些取值？（Day 1.4）
4. AAA 模式指什么？（Day 2.4）
5. pytest 的测试文件、测试函数命名约定是什么？异常用例怎么写？（Day 2.2 / 2.4）
6. 什么样的代码需要 Mock？`MagicMock` 能帮你断言什么？（Day 3）
7. 分支覆盖率 100% 意味着没有 bug 吗？为什么？（Day 4.3）
8. TDD 的"红绿重构"是什么顺序？它倒逼你先想清楚什么？（Day 4.4）
9. 同事改了一行业务代码，你应该立刻做什么？（Day 5.3：跑测试）

## 5.5 进阶路线图

完成 5 天后，按需深入：

| 方向 | 学什么 |
|------|--------|
| 用例设计进阶 | 正交分析法、错误推测法；《单元测试的艺术》（Roy Osherove） |
| pytest 深入 | fixture 体系、conftest.py、插件生态 |
| Web 开发测试 | Flask/Django/FastAPI 的接口测试（pytest + TestClient） |
| TDD 深入 | 在真实小项目中坚持"先写测试"两周 |
| 测试遗留代码 | 《修改代码的艺术》（Working Effectively with Legacy Code） |
| 上一层测试 | 集成测试 → E2E 自动化，见 `docs/auto/自动化测试学习方案.md` |

---

## 附录 A：术语速查表

| 术语 | 含义 |
|------|------|
| SUT | System Under Test，被测系统/被测单元 |
| 断言（Assert） | 判定"实际结果 == 预期结果"的语句 |
| 回归（Regression） | 以前好的功能被新改动弄坏 |
| 回归测试 | 防止回归的测试，通常全量反复执行 |
| 测试替身（Test Double） | 顶替真实依赖的假对象（Stub/Spy/Mock 等） |
| 依赖注入 | 把依赖作为参数传入，方便测试时替换 |
| 夹具（Fixture） | 为用例准备/清理环境的机制（如 pytest 的 fixture、monkeypatch） |
| 覆盖率（Coverage） | 测试执行到的代码占比（行/分支/函数） |
| TDD | 测试驱动开发：先写测试，再写实现 |
| 冒烟测试 | 最基础的一组快速用例，先确认"系统没崩" |

## 附录 B：常见问题

**Q：测试失败了，先怀疑实现还是先怀疑测试？**
刚写的测试就红：多半是实现有 bug，或你对预期本身理解错了（这正是 TDD"红"阶段的价值）。一直绿的老测试突然红：多半是有人改坏了实现——测试正在干它的本职工作。

**Q：要不要追求 100% 覆盖率？**
核心业务逻辑（借书规则、罚款计算）尽量分支 100%；界面胶水代码不必强求。**永远不要为凑覆盖率写没有断言的假测试。**

**Q：到底什么值得写单测？**
有逻辑、有分支、有边界的东西（计算、规则、解析、状态判断）；不值得测的：简单的属性赋值、纯配置。拿不准时问自己：这块代码坏了，测试能比人更早发现吗？

**Q：`ModuleNotFoundError: No module named 'borrow'` 怎么办？**
检查两点：① 是否在 `pytest ut` 时丢掉了目录参数；② 被测模块和 `test_*.py` 是否在同一目录（本方案约定都在 `ut/` 下）。

## 附录 C：推荐资源

- pytest 官方文档（入门篇足够 5 天使用）：https://docs.pytest.org/en/stable/getting-started.html （社区有中文翻译版，搜索"pytest 中文文档"）
- unittest.mock 标准库文档：https://docs.python.org/zh-cn/3/library/unittest.mock.html （Python 官方中文文档）
- 书：《单元测试的艺术》（Roy Osherove）——学完这 5 天后读第 1~4 章；《Python Testing with pytest》（Brian Okken，英文）
- 对照阅读：`docs/auto/自动化测试学习方案.md`（同样基于 pytest 生态，无缝衔接）
