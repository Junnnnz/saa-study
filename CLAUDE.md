# AWS SAA 备考工具 — CLAUDE.md

## 项目概述

这是一个**单文件 HTML 备考工具**，部署在 GitHub Pages 上，通过浏览器直接打开使用。
- 文件：`index.html`（全部逻辑 CSS JS 合并在一个文件里）
- 数据持久化：使用 `window.storage` API 跨 session 保存答题记录
- 部署：GitHub Pages，push 即生效

## 使用者信息

- 用英文参加 AWS SAA-C03 考试
- 正在学习 Stephane Maarek 的 Udemy 课程
- 目前进度：IAM → EC2 → EBS/EFS → 可扩展性/HA → Load Balancer → Auto Scaling Group（最新）
- 下一步：S3 章节

## 技术架构

```
index.html
├── <style>          CSS 变量 + 组件样式（约 400 行）
├── HTML skeleton    侧边栏导航 + 主内容区 + 移动端底部导航
└── <script>
    ├── QS[]         题库数组（每题含中英文双语）
    ├── Store        答题记录存储（window.storage）
    ├── go(id)       页面路由函数
    ├── 题库引擎      beginQuiz / renderQuiz / pickAns / showResult
    ├── 知识页函数    pgIAM / pgEBS / pgELB / pgASG / ...
    └── 对比页函数    pgCompare（含 6 个 tab）
```

## 添加新知识页的规范

每个知识页是一个函数 `pgXXX()` 返回 HTML 字符串。结构：
```javascript
function pgXXX(){return`
  ${back()}                    // 返回按钮（固定格式）
  <div class="pg-hdr">...</div>  // 页面标题
  <div class="s-card">           // 内容卡片
    <h3>标题</h3>
    ${bullet([...])}             // 列表内容
    <div class="tbl-wrap">       // 表格
    ${trapdiv('red','高频','陷阱标题','说明')}  // 考试陷阱
  </div>
`}
```

注册新页面到路由：在 `go()` 函数的 `renders` 对象里加 `{id: pgXXX}`

## 添加题目的规范

在 `QS[]` 数组末尾添加，结构：
```javascript
{
  id: 's3_01',              // 格式：主题_序号
  topic: 'S3',              // 主题名（要和侧边栏/题库选项一致）
  diff: 'high',             // 'high' / 'mid' / 'low'
  text: '题目（中文，可含 <em> <strong> <span class="code-inline">）',
  en: '题目（英文，可选但推荐）',
  ch: ['A选项', 'B选项', 'C选项', 'D选项'],
  ench: ['A EN', 'B EN', 'C EN', 'D EN'],  // 英文选项（可选）
  ans: 1,                   // 正确答案索引（0-based）
  exp: '解析（支持 HTML）',
  ww: '其他选项为何错（可选）',
  rule: '📌 记忆口诀（一句话总结）'
}
```

## 开发规则

1. **不要拆分文件**：保持单文件 HTML，方便 GitHub Pages 部署和离线使用
2. **修改前先 grep 确认位置**，避免重复内容
3. **JS 语法检查**：修改 QS[] 或 JS 逻辑后，用 `node --check` 验证
4. **保持双语**：知识页的服务名、技术术语要同时标注中英文
5. **不删除已有内容**：只增量添加，保留用户的答题记录逻辑

## 待添加的内容（按优先级）

### 下一章：S3
- S3 基础（Bucket / Object / Key / Region）
- S3 存储类型（Standard / IA / Glacier / ...）
- S3 安全（Bucket Policy / ACL / Encryption）
- S3 高级功能（Versioning / Replication / Lifecycle）
- S3 相关题目（至少 15 道）

### 对比页待补充
- S3 vs EFS vs EBS（存储三选一已有但可扩展 S3）
- S3 存储类型对比表

## 侧边栏新增入口规范

在 `sidebar` 的对应 `sb-section` 里加按钮：
```html
<button class="sb-item" onclick="go('s3',this)">
  <span class="si-dot" style="background:#E8501A"></span>S3 基础
</button>
```

同时在首页 `chapters` 数组的对应 label 下加 svc-card 项目。

## 部署

```bash
# 初次设置
git init
git remote add origin https://github.com/YOUR_USERNAME/saa-study.git
git add index.html CLAUDE.md README.md
git commit -m "init: SAA study tool"
git push -u origin main

# 之后每次更新
git add index.html
git commit -m "add: S3 chapter + 15 quiz questions"
git push

# GitHub repo Settings → Pages → Branch: main / root → Save
# 访问 https://YOUR_USERNAME.github.io/saa-study
```
