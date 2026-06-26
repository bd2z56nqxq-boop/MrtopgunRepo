# CC Multimodal Toolkit

> 给 Claude Code 装上眼睛和画笔：一次安装，三种能力。

## 能力矩阵

| 能力 | 触发方式 | 引擎 | 耗时 |
|------|----------|------|------|
| 🔍 读图分析 | `/cgmb` 或 "看下截图" | Gemini 2.5 Flash | 10-30s |
| 🎨 AI 生图 | `/codex:rescue` 或 "生成xxx图片" | gpt-image-2 | 30-60s |
| 📸 截图反馈 | "这里不对" 等引用 | 自动检测 → CGMB | 10-30s |

## 安装

### 1. 安装本插件

```bash
claude plugin marketplace add openai/codex-plugin-cc
claude plugin install codex@openai-codex --scope user
claude plugin install cc-multimodal-toolkit@bd2z56nqxq-boop/MrtopgunRepo --scope user
```

### 2. 安装 CGMB（读图能力）

```bash
npm install -g claude-gemini-multimodal-bridge --ignore-scripts
npm install -g aistudio-mcp-server
npm install -g @google/gemini-cli
```

### 3. 配置 API Key

在 `C:/Users/Administrator/.env` 中写入：
```env
AI_STUDIO_API_KEY=你的Google_AI_Studio_Key
GEMINI_API_KEY=你的Gemini_API_Key
```

去 https://aistudio.google.com/app/apikey 创建 Key。

### 4. 配置 Codex（生图能力）

```bash
codex login    # 交互式 OAuth（需 ChatGPT 订阅）
```

### 5. 配置截图目录

确保使用 `Win + PrtScn` 截图（自动保存到 `Pictures/Screenshots/`）。

## 使用

### 读图

```
/cgmb                           # 读最新截图
/cgmb 描述这张图                  # 读最新截图 + 描述
/cgmb D:/photo.png 这是什么       # 指定图片分析
/cgmb chat 今天AI圈有什么新闻      # Gemini 网页搜索
```

### 生图

```
/codex:rescue 赛博朋克城市夜景海报
```
或直接说：
```
生成一张秋天红叶林间的照片
```

### 截图反馈

截图（Win+PrtScn）后直接说：
```
这里逻辑不对，帮我改
看下这个报错
这个界面需要优化
```
Claude Code 自动读取最新截图并结合你的文字处理。

## 架构

```
┌──────────────────────────────────────────┐
│            Claude Code 会话                │
│                                           │
│  ┌─────────────┐   ┌─────────────┐       │
│  │ CGMB/Gemini │   │ Codex/OpenAI │       │
│  │  读图·分析   │   │  生图·编辑   │       │
│  └──────┬──────┘   └──────┬──────┘       │
│         │                 │               │
│  ┌──────▼─────────────────▼───────────┐  │
│  │       自动截图检测引擎              │  │
│  │  Screenshots/ → 最新文件 → CGMB    │  │
│  └────────────────────────────────────┘  │
└──────────────────────────────────────────┘
```

## 踩坑记录

- **CGMB postinstall 在 Windows 上失败**：用 `--ignore-scripts` 后手动补装依赖
- **Codex WebSocket 连接超时**：确保代理规则覆盖 `chatgpt.com` 域名
- **对话框拖图无效**：终端不支持二进制粘贴，用 Win+PrtScn 截图替代
- **网络诊断**：`curl -s -o NUL -w "%{http_code}" --max-time 10 https://chatgpt.com` 返回 000 表示代理故障

## License

MIT
