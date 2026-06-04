# 计组刷题助手 · 项目介绍书

## 项目概要

一个纯前端刷题 SPA，用于计算机组成原理备考。支持单选/判断/填空/简答/计算五种题型，可在手机端和 PC 端使用。

**项目性质**：纯静态 HTML + JS + CSS，无后端、无构建工具。可直接 `file://` 打开或部署到 GitHub Pages。

---

## 文件结构

```
├── index.html              ← ★ 主力手机版（GitHub Pages 发布）
├── quiz-mobilenew.html     ← index.html 的副本，独立手机端入口
├── quiz-mobile.html        ← 【已停止维护】历史版本
├── quiz-pc.html            ← PC 桌面版（大屏优化，键盘快捷键）
├── CLAUDE.md               ← 本文件
├── CHANGELOG.md
├── data/
│   ├── jz_ch1.js ~ jz_ch4.js  ← 题库数据文件
│   └── raw/                    ← 原始文本题库（不上传 GitHub）
├── extract_data.py          ← 从 raw/ 提取题库的脚本（本地工具）
└── .gitignore
```

### 三个版本的定位

| 文件 | 端 | 数据源 | 说明 |
|------|-----|--------|------|
| `index.html` | 手机 | data/*.js | GitHub Pages 发布版，多科目，主力维护 |
| `quiz-mobilenew.html` | 手机 | data/*.js | index.html 副本，需手动同步更新 |
| `quiz-mobile.html` | 手机 | 内嵌在 HTML | 旧版已弃用 |
| `quiz-pc.html` | PC | data/*.js | 大屏优化，独立维护 |

**同步策略**：改 `index.html` → 复制一份到 `quiz-mobilenew.html` → 一起提交。

---

## 架构要点

### 数据加载（index.html / quiz-pc.html）

题库数据通过 `<script>` 标签动态加载 `data/*.js` 文件，兼容 `file://` 协议：

```javascript
// data/jz_ch1.js 内部调用
_QDATA("jz", "ch1", { id: "ch1", name: "第一章", questions: [...] });
```

`index.html` 的 `_QDATA()` 函数将数据注册到全局 `QUESTION_BANK` 对象：
```javascript
QUESTION_BANK = {
  "jz": {
    "ch1": { id, name, questions },
    "ch2": { id, name, questions },
    ...
  }
}
```

### 科目与章节注册

内嵌在 `index.html` 的 `_MANIFEST_DATA` 变量中：

```javascript
const _MANIFEST_DATA = {
  subjects: [
    { id: "jz", name: "计算机组成原理", icon: "🖥️",
      chapters: [
        { id: "ch1", name: "第一章 计算机系统概述" },
        ...
      ]
    }
  ]
};
```

**添加新科目/章节**：改 `_MANIFEST_DATA`，新建 `data/<科目id>_<章节id>.js` 即可，无需改 HTML 逻辑。

### localStorage 键名

每个版本使用不同的键名前缀，数据互不干扰：

| 版本 | 错题 | 收藏 | 历史 | 最后科目 |
|------|------|------|------|----------|
| index.html | `ccw_科目ID` | `ccs_科目ID` | `cch_科目ID` | `cc_last_sub` |
| quiz-mobilenew.html | 同上 | 同上 | 同上 | 同上 |
| quiz-pc.html | `pcw_科目ID` | `pcs_科目ID` | `pch_科目ID` | `pc_last_sub` |

**注意**：`index.html` 和 `quiz-mobilenew.html` 共享 localStorage，进度互通。

---

## 题型系统

五种题型，定义在 `TYPE_ORDER`、`TYPE_LABELS`、`TYPE_CLASSES` 中：

| 类型 key | 显示名 | CSS 类 |
|----------|--------|--------|
| `danxuan` | 单选题 | `q-type-dx` |
| `panduan` | 判断题 | `q-type-pd` |
| `tiankong` | 填空题 | `q-type-tk` |
| `jianda` | 简答题 | `q-type-jd` |
| `jisuan` | 计算题 | `q-type-js` |

题目数据结构：
```javascript
{
  id: "ch1_1",           // 必须唯一
  type: "danxuan",       // 对应 TYPE_ORDER
  question: "题干文本",
  options: ["A. 选项A", "B. 选项B", ...],  // 选择题才有
  answer: "D"            // 选择题用字母，填空题用文本，简答/计算用多行文本
}
```

---

## 功能清单

- [x] 五种题型刷题（单选/判断/填空/简答/计算）
- [x] 章节刷题、全部刷题、随机刷题
- [x] 错题本、收藏（⭐）
- [x] 题型筛选（按题型过滤题目）
- [x] 题号快速跳转（输入数字回车）
- [x] 触屏滑动切换题目（左滑下一题/右滑上一题）
- [x] 答案即时反馈 + 参考答案展示
- [x] 答题历史记录（localStorage）
- [x] 多科目支持
- [x] 题库浏览页（统计总览）
- [x] PC 版键盘快捷键（← → 翻题、1~4 选选项、Space 看答案等）

---

## 维护提示

### 添加题目
1. 打开 `data/jz_chX.js`
2. 在对应章节的 `questions` 数组中添加新题目对象
3. 确保 `id` 不重复（约定：`chX_数字`、`chX_f数字`（填空）、`chX_s数字`（简答）、`chX_c数字`（计算））

### 添加新章节
1. 新建 `data/jz_ch5.js`，格式参考已有文件
2. 修改 `index.html` 和 `quiz-pc.html` 中的 `_MANIFEST_DATA`，在对应科目的 `chapters` 中添加条目

### 添加新科目
1. 新建 `data/<新id>_ch1.js` 等文件
2. 在 `_MANIFEST_DATA.subjects` 中添加新科目对象
3. 三个 HTML 文件都要改

### 滑动效果
```javascript
// 关键参数（在 touchend 事件中）
const threshold = 50;      // 最少滑动 px
const maxTime = 400;       // 最长滑动 ms
const followFactor = 0.4;  // 跟随系数
const slideOutTime = 140;  // 滑出动画 ms
const slideInTime = 220;   // 滑入动画 ms
```

---

## 常见问题

**Q: 为什么用 `<script>` 加载而不是 fetch JSON？**
兼容 `file://` 协议（本地直接打开 HTML）。fetch 在 `file://` 下会被 CORS 阻止。

**Q: 两个手机版进度冲突吗？**
`index.html` 和 `quiz-mobilenew.html` 共用 localStorage 键名，共享进度。`quiz-pc.html` 独立。

**Q: 如何在手机上测试？**
用手机浏览器打开 `index.html` 即可（本地或 GitHub Pages）。

---

## 更新日志

见 `CHANGELOG.md`。
