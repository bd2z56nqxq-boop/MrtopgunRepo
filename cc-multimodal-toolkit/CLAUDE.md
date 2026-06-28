# CC 多模态工作台 — 行为规则

安装本插件后，Claude Code 自动获得以下行为：

## 截图反馈工作流

当用户的消息中包含对"图"、"截图"、"这个"、"这里"、"看下"、"你看"等图像引用，但没有明确给出文件路径时：

1. 自动查找截图目录中最新的截图
   - **Windows PowerShell:**
     ```powershell
     $dir = Join-Path $env:USERPROFILE "Pictures\Screenshots"
     Get-ChildItem -LiteralPath $dir -File |
       Sort-Object LastWriteTime -Descending |
       Select-Object -First 1 -ExpandProperty FullName
     ```
   - **macOS / Linux:**
     ```bash
     ls -lt ~/Desktop/ | head -2
     ```
2. 用 CGMB (`cgmb analyze`) 读取截图内容
3. 结合用户的文字反馈和要求进行后续处理

**触发示例：**
- "这里不对，改一下" → 自动读最新截图 → 结合反馈处理
- "CGMB解读" → 直接读最新截图并分析
- "看下这张图 D:/photo.png" → 分析指定图片

## 稳定入口：`/cgmb`

`/cgmb` 是本插件的核心命令，支持：
- `/cgmb` — 读最新截图
- `/cgmb <描述>` — 读最新截图 + 指定分析角度
- `/cgmb <图片路径> <描述>` — 分析指定图片
- `/cgmb chat <问题>` — Gemini 网页搜索

## 生图规则

- AI 生图依赖 OpenAI Codex 插件（需单独安装）
- 用户说"生成xxx图片" → 使用 Codex CLI (`codex exec`) 生图
- 用户使用 `/codex:rescue` 命令 → 同上
- 生图默认输出到 `~/output/images/`

## 环境变量

以下环境变量需预先配置：
- `AI_STUDIO_API_KEY` — Google AI Studio API Key（Gemini 读图）
- `GEMINI_API_KEY` — Gemini API Key（网页搜索）
- Codex 登录状态 — `codex login`（生图）
