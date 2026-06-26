---
description: 用 Codex gpt-image-2 生成图片
argument-hint: "图片描述（如：一只坐在窗台上的猫）"
allowed-tools: Bash
---

## Generate — AI 生图

**用户输入：** `$ARGUMENTS`

**前置条件：** Codex CLI 已安装并登录（`codex login`），Codex 插件已启用。

**执行逻辑：**

直接调用 Codex CLI 生成图片，在 prompt 末尾追加保存路径：

```bash
codex exec "$ARGUMENTS。Save to C:/Users/Administrator/output/images/<descriptive-filename>.png" --yolo
```

**文件名规则：** 根据用户描述提取关键词，用英文命名，小写连字符分隔（如 `sunset-city.png`）。

**输出：** 告知用户生成结果和文件路径，展示图片。
