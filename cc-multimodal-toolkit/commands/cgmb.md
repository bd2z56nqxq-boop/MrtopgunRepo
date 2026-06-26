---
description: 用 Gemini 读图分析——截图、指定图片、或网页搜索
argument-hint: "[图片路径] 描述/问题"
allowed-tools: Bash
---

## CGMB — Gemini 多模态读图

**用户输入：** `$ARGUMENTS`

**前置条件：** `cgmb` CLI 已安装（`npm install -g claude-gemini-multimodal-bridge`），AI_STUDIO_API_KEY 和 GEMINI_API_KEY 已配置。

**执行逻辑：**

### 1. 用户提供了文件路径
如果 $ARGUMENTS 中有图片路径（`.png`、`.jpg`、`.jpeg`、`.webp`、`.gif`、`.bmp`），执行：
```bash
cgmb analyze "<文件路径>" --prompt "<用户的描述或问题>"
```

### 2. 用户没给文件路径 → 自动找最新截图
```bash
ls -lt "C:/Users/Administrator/Pictures/Screenshots/" | head -2
```
然后对最新文件执行：
```bash
cgmb analyze "<最新截图路径>" --prompt "<用户的描述或问题>"
```

### 3. 用户以 chat/搜索/search 开头
```bash
cgmb chat "<用户问题>"
```

**输出：** 将 CGMB 的分析结果原文返回，不要改写或总结。
