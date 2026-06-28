---
description: 用 Gemini 读图分析——截图、指定图片、或网页搜索
argument-hint: "[chat <问题>] [图片路径] [描述]"
allowed-tools: Bash
---

## CGMB — Gemini 多模态读图 & 网页搜索

**用户输入：** `$ARGUMENTS`

**前置条件：** `cgmb` CLI 已安装，AI_STUDIO_API_KEY 和 GEMINI_API_KEY 已配置。

---

### 0. 路由判断（先看类型，再看平台）

根据 `$ARGUMENTS` 开头决定走哪条路径：

| 输入 | 路径 |
|------|------|
| 以 `chat` 开头 | → **路径 A：网页搜索** |
| 以 `.png` `.jpg` `.jpeg` `.gif` `.webp` `.bmp` 结尾 或 以盘符开头（如 `D:/` `C:/` `/` `~`） | → **路径 B：指定图片分析** |
| 空 或 纯描述文字 | → **路径 C：自动找截图 → 图片分析** |

**平台检测：** 先检查当前 shell 类型。若 `$env:OS` 或 `$env:ComSpec` 存在则为 Windows（用 PowerShell/cgmb.cmd），否则为 macOS/Linux（用 bash/cgmb）。

---

### 路径 A：网页搜索

当 `$ARGUMENTS` 以 `chat` 开头时，去除前缀 `chat`，用 Gemini CLI 执行网页搜索：

**Windows PowerShell:**
```powershell
$query = "$ARGUMENTS" -replace '^chat\s*', ''
cgmb.cmd chat $query
```

**macOS / Linux:**
```bash
QUERY="${ARGUMENTS#chat }"
cgmb chat "$QUERY"
```

处理完后直接返回结果，不走后面的图片分析。

---

### 路径 B & C：图片分析（盘存优先，杜绝遗漏）

**核心策略：不用「描述」用「盘存」——让 Gemini 逐像素抄录，而非概括。**

#### 1. 确定图片路径

**路径 B — 用户已提供图片路径：** 直接使用 `$ARGUMENTS` 中的路径。

**路径 C — 自动找截图：**

**Windows PowerShell:**
```powershell
$dir = Join-Path $env:USERPROFILE "Pictures\Screenshots"
if (-not (Test-Path $dir)) {
  Write-Error "截图目录不存在: $dir。请用 Win+PrtScn 截图后再试。"
  exit 1
}
$files = Get-ChildItem -LiteralPath $dir -File -Include *.png,*.jpg,*.jpeg,*.gif,*.webp,*.bmp |
  Sort-Object LastWriteTime -Descending |
  Select-Object -First 1
if (-not $files) {
  Write-Error "截图目录为空: $dir。请用 Win+PrtScn 截图后再试。"
  exit 1
}
Write-Output $files.FullName
```

**macOS:**
```bash
# 优先搜 Screenshots 目录，回退到 Desktop
for DIR in "$HOME/Pictures/Screenshots" "$HOME/Desktop"; do
  if [ -d "$DIR" ]; then
    IMG=$(ls -t "$DIR"/*.{png,jpg,jpeg,gif,webp,bmp} 2>/dev/null | head -1)
    if [ -n "$IMG" ]; then
      echo "$IMG"
      break
    fi
  fi
done
if [ -z "$IMG" ]; then
  echo "未找到截图。请截图后再试。" >&2
  exit 1
fi
```

**Linux:**
```bash
for DIR in "$HOME/Pictures/Screenshots" "$HOME/Pictures" "$HOME/Desktop"; do
  if [ -d "$DIR" ]; then
    IMG=$(ls -t "$DIR"/*.{png,jpg,jpeg,gif,webp,bmp} 2>/dev/null | head -1)
    if [ -n "$IMG" ]; then
      echo "$IMG"
      break
    fi
  fi
done
if [ -z "$IMG" ]; then
  echo "未找到截图。请截图后再试。" >&2
  exit 1
fi
```

#### 2. 执行完整盘存（两步法）

**第一步：完整盘存**（防止遗漏）

先确认 cgmb 命令名（Windows 用 `cgmb.cmd`，其他用 `cgmb`），然后：

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
