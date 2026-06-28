# CC Multimodal Toolkit

> 给 Claude Code 装上眼睛和画笔：Gemini 读图 + Codex 生图 + 截图工作流

## 能力矩阵

| 能力 | 触发方式 | 引擎 | 耗时 |
|------|----------|------|------|
| 🔍 读图分析 | `/cgmb` 命令 | Gemini 2.5 Flash | 10-30s |
| 🔍 网页搜索 | `/cgmb chat <问题>` | Gemini | 5-15s |
| 🎨 AI 生图 | `/codex:rescue` / "生成xxx图片" | gpt-image-2 | 30-60s |

## 安装

### 0. 前置条件

- 已安装 [Claude Code](https://claude.ai/download)
- 已安装 Node.js / npm
- **Windows 用户建议在 PowerShell 中使用 `.cmd` 后缀**（如 `npm.cmd`、`claude.cmd`），避免 PowerShell 执行策略问题

### 1. 安装本插件

**macOS / Linux / Git Bash:**

```bash
claude plugin marketplace add bd2z56nqxq-boop/MrtopgunRepo
claude plugin install cc-multimodal-toolkit@mrtopgun-plugins --scope user
```

**Windows PowerShell:**

```powershell
claude.cmd plugin marketplace add bd2z56nqxq-boop/MrtopgunRepo
claude.cmd plugin install cc-multimodal-toolkit@mrtopgun-plugins --scope user
```

### 2. 安装 CGMB 依赖（读图能力）

**macOS / Linux / Git Bash:**

```bash
npm install -g claude-gemini-multimodal-bridge --ignore-scripts
npm install -g aistudio-mcp-server
npm install -g @google/gemini-cli
```

**Windows PowerShell:**

```powershell
npm.cmd install -g claude-gemini-multimodal-bridge --ignore-scripts
npm.cmd install -g aistudio-mcp-server
npm.cmd install -g @google/gemini-cli
```

### 3. 配置 API Key 与登录

在用户目录下的 `.env` 文件（或系统环境变量）中配置：

```env
AI_STUDIO_API_KEY=你的Google_AI_Studio_Key
GEMINI_API_KEY=你的Gemini_API_Key
```

> 去 https://aistudio.google.com/app/apikey 创建 Key（免费）。

然后运行验收：

**macOS / Linux / Git Bash:**

```bash
cgmb auth --interactive
cgmb setup-mcp
cgmb verify
claude auth
```

**Windows PowerShell:**

```powershell
cgmb.cmd auth --interactive
cgmb.cmd setup-mcp
cgmb.cmd verify
claude.cmd auth
```

### 4. 安装 Codex 插件（生图能力）

安装 OpenAI Codex 插件实现 AI 生图：

**macOS / Linux / Git Bash:**

```bash
claude plugin marketplace add openai/codex-plugin-cc
claude plugin install codex@openai-codex --scope user
codex login
```

**Windows PowerShell:**

```powershell
claude.cmd plugin marketplace add openai/codex-plugin-cc
claude.cmd plugin install codex@openai-codex --scope user
codex.cmd login
```

### 5. 配置截图目录（可选）

本插件的 `/cgmb` 命令会自动查找截图目录。确保截图工具将图片保存到系统的 `Pictures/Screenshots/` 目录（Windows 使用 `Win + PrtScn` 即自动保存至此）。

## 使用

### 读图（稳定入口：`/cgmb`）

```
/cgmb                           # 读最新截图
/cgmb 描述这张图                  # 读最新截图 + 描述
/cgmb D:/photo.png 这是什么       # 指定图片分析
/cgmb chat 今天AI圈有什么新闻      # Gemini 网页搜索
```

> **注意：** 当前稳定入口是 `/cgmb` 命令。直接说"这里不对""看下截图"等自然语言自动触发能力，取决于你的 `CLAUDE.md` 中是否配置了截图检测规则（见下方"进阶配置"）。

### 生图

```
/codex:rescue 赛博朋克城市夜景海报     # Codex 生图命令
```
或直接说：
```
生成一张秋天红叶林间的照片
```

### 进阶配置（可选）

如果你希望 Claude Code 在你提到"截图""这里不对"时自动读图，可以在你的用户级 `CLAUDE.md` 中添加截图检测规则：

```markdown
## 截图反馈工作流

当用户的消息中包含对"图"、"截图"、"这个"、"这里"、"看下"等图像引用，但没有明确给出文件路径时：

1. 自动查找截图目录中最新的截图
   - Windows PowerShell:
     $dir = Join-Path $env:USERPROFILE "Pictures\Screenshots"
     Get-ChildItem -LiteralPath $dir -File |
       Sort-Object LastWriteTime -Descending |
       Select-Object -First 1 -ExpandProperty FullName
   - macOS: ls -lt ~/Desktop/ | head -2
2. 用 CGMB 读取截图内容
3. 结合用户的文字反馈进行后续处理
```

## 架构

```
┌──────────────────────────────────────────┐
│            Claude Code 会话                │
│                                           │
│  ┌─────────────┐   ┌─────────────┐       │
│  │ CGMB/Gemini │   │ Codex/OpenAI │       │
│  │  读图·搜索   │   │  生图·编辑   │       │
│  └──────┬──────┘   └──────┬──────┘       │
│         │                 │               │
│  ┌──────▼─────────────────▼───────────┐  │
│  │           /cgmb 命令入口            │  │
│  │  Screenshots/ → 最新文件 → CGMB    │  │
│  └────────────────────────────────────┘  │
└──────────────────────────────────────────┘
```

## 踩坑记录

- **CGMB postinstall 在 Windows 上失败**：用 `--ignore-scripts` 后手动补装依赖
- **Codex WebSocket 连接超时**：确保代理规则覆盖 `chatgpt.com` 域名
- **截图路径不对**：确保使用了 `$env:USERPROFILE` 而非硬编码用户名
- **Windows 命令找不到**：使用 `.cmd` 后缀（`npm.cmd`、`claude.cmd`、`cgmb.cmd`）
- **网络诊断**：`curl -s -o NUL -w "%{http_code}" --max-time 10 https://chatgpt.com` 返回 000 表示代理故障

## License

MIT
