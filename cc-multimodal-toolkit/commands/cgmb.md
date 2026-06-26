---
description: 用 Gemini 读图分析——截图、指定图片、或网页搜索
argument-hint: "[图片路径] 描述/问题"
allowed-tools: Bash
---

## CGMB — Gemini 多模态读图

**用户输入：** `$ARGUMENTS`

**前置条件：** `cgmb` CLI 已安装，AI_STUDIO_API_KEY 和 GEMINI_API_KEY 已配置。

**关键原则：截图可能包含用户标注（红框、红圈、箭头、画笔、手写批注），这些标注比正文更重要。分析截图时必须优先识别标注。**

### 1. 用户提供了文件路径
如果 $ARGUMENTS 中有图片路径（`.png`、`.jpg`、`.jpeg`、`.webp`、`.gif`、`.bmp`），执行：
```bash
cgmb analyze "<文件路径>" --prompt "仔细查看这张图。
1. 优先找出所有标注：红框、红圈、箭头、画笔划痕、手写文字、高亮标记等。
2. 对每个标注，说明它的位置、形状、指向的内容。
3. 读取图中所有可见文字（标注旁边批注、UI文字、代码等）。
4. 将标注内容与被标注区域关联，还原用户反馈意图。
用户附加说明：$ARGUMENTS"
```

### 2. 用户没给文件路径 → 自动找最新截图
```bash
ls -lt "C:/Users/Administrator/Pictures/Screenshots/" | head -2
```
然后对最新文件执行（使用上述同款标注优先 prompt）：
```bash
cgmb analyze "<最新截图路径>" --prompt "仔细查看这张截图。
1. 优先找出所有标注：红框、红圈、箭头、画笔划痕、手写文字、高亮标记等。
2. 对每个标注，说明它的位置、形状、指向的内容。
3. 读取图中所有可见文字（标注旁边批注、UI文字、代码等）。
4. 将标注内容与被标注区域关联，还原用户反馈意图。
用户附加说明：$ARGUMENTS"
```

### 3. 用户以 chat/搜索/search 开头
```bash
cgmb chat "$ARGUMENTS"
```

**输出：** 将 CGMB 的分析结果原文返回，不要改写或总结。标注信息优先呈现。
