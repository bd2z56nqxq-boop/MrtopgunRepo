---
description: 用 Gemini 读图分析——截图、指定图片、或网页搜索
argument-hint: "[图片路径] 描述/问题"
allowed-tools: Bash
---

## CGMB — Gemini 多模态读图（盘存优先，杜绝遗漏）

**用户输入：** `$ARGUMENTS`

**前置条件：** `cgmb` CLI 已安装，AI_STUDIO_API_KEY 和 GEMINI_API_KEY 已配置。

**核心策略：不用「描述」用「盘存」——让 Gemini 逐像素抄录，而非概括。**

### 1. 自动找截图（用户未提供路径时）

**Windows PowerShell:**
```powershell
$dir = Join-Path $env:USERPROFILE "Pictures\Screenshots"
Get-ChildItem -LiteralPath $dir -File |
  Sort-Object LastWriteTime -Descending |
  Select-Object -First 1 -ExpandProperty FullName
```

**macOS / Linux:**
```bash
ls -lt ~/Desktop/ | head -2
```

### 2. 执行完整盘存（两步法）

**第一步：完整盘存**（防止遗漏）
```bash
cgmb analyze "<截图路径>" --prompt "请逐像素级仔细查看这张截图，按以下格式输出完整盘存：

【标注/批注】（最重要，优先列出）
- 每个红框/红圈/箭头/画笔/高亮的位置、形状、指向哪个UI元素
- 每个手写/叠加文字的原文内容、位置、颜色

【界面文字】（逐区域完整抄录）
- 标题栏、菜单、按钮、输入框、标签页、状态栏的所有可见文字
- 弹窗/对话框/错误提示的完整内容
- 代码区域的完整行（如有代码）

【界面结构】
- 这是什么应用/页面的什么页面/区域
- 各UI组件的层级关系

注意：不要概括，不要跳过。每个字都要抄下来。"
```

**第二步：结合用户意图**
将盘存结果与用户输入（`$ARGUMENTS`）结合分析，输出用户问题的针对性回答。

**输出：** CGMB 结果原文返回，不改写不总结。
