---
name: reduce-ai-rate
description: Use when a user provides a Chinese academic paper (.docx) with high AI detection rate and needs to lower it below a target threshold (e.g. <25%). The user explicitly mentions "AI率" or "降AI" or "AIGC检测" or their university flagged the paper for AI-generated content.
---

# Reduce AI Rate (论文降AI率)

## Overview
Systematic workflow for reducing AI detection rate in Chinese academic papers. Reads a .docx file, classifies paragraphs, applies targeted rewriting based on user's style preference, and outputs a clean version. The core insight: **commercial AI detectors (知网/维普/PaperYY)抓的不是"AI概率"，而是"AI写作特征"——句长均匀、排比密集、连接词模式化、缺乏个人观点。消除这些特征比拟合任何检测模型更有效。**

## Workflow

```
用户输入.docx路径 + 选择风格(专业/平衡/口语化)
  → python-docx提取纯文本
  → 分类段落: keep(封面/声明/目录/图题/参考文献) vs rewrite(正文)
  → 按所选风格的策略改写
  → 输出Markdown对照稿 + 纯文本到目标目录
  → 自检(规则扫描AI特征密度)
  → 告诉用户结果 + 建议用学校工具终检
```

## Style Options

Present the three options to the user with expected outcomes:

| 风格 | 改写策略 | 预期降AI率(v3参考) | 学术感保留 |
|------|---------|:-----------------:|:---------:|
| **专业版** | 仅替换AI高频词(本文→此次/该研究)、调整句首、打散排比；不动学术结构和术语，不加入口语表达 | 95%→85% (降~10%) | ★★★★★ |
| **平衡版** | 替换高频词+打散排比+适度加个人视角；保留学术语气但降低结构工整度；部分连接词替换为自然表达 | 95%→75% (降~20%) | ★★★★ |
| **口语化版** | 拆长句为短句、加个人视角(我觉得/我发现/我的感受)、口语连接词(不过说实话/说到底)、减四字词组堆砌；接受一定非正式感 | 95%→65% (降~30%) | ★★★ |

注意：以上数值基于v3模型。不同学校工具数值不同，但**相对比例一致**——口语化版一定比专业版降得多。

## Detailed Workflow

### Step 1: Read .docx
```python
from docx import Document
doc = Document(path)
```
Extract all paragraphs with index, style, and text.

### Step 2: Classify Paragraphs
Keep (don't rewrite):
- 封面信息：学号、姓名、学院、专业、班级、指导教师、完成日期
- 诚信声明、知识产权声明
- 目录
- 图题（图1.xxx、表1.xxx）及图片来源说明
- 参考文献列表
- 英文摘要（只做微调）

Rewrite: Everything else (摘要、引言、正文各章节、总结、致谢)

### Step 3: Apply Rewriting Based on Style

**专业版手法：**
- "本文" → "本研究"/"此次设计"
- "目前" → "现阶段"/去掉
- "随着" → 直接陈述
- "因此" → "基于此"/"鉴于此"  
- 排比句拆成2-3句话
- 段落首句多样化
- 保留：学术术语、四字专业词、正式语气

**平衡版手法：**
- 以上全部 + 适度加入"我/我们"视角
- "首先/其次/最后" → 去掉或用自然过渡
- "综上所述" → "总的来看"/去掉
- 部分长句拆短
- 加入少量个人观察句
- 保留大部分学术语气

**口语化版手法：**
- 以上全部 + 大量加入口语表达
- "本文" → "我这回/这次"
- "因此" → "所以说"
- "然而" → "不过说实话"
- 长句全部拆成短句（每句<40字）
- 加入"我觉得""我发现""我的感受"
- 段落结构完全打散（不保留总-分-总）
- 减少四字成语和学术词组

### Step 4: Output
- Markdown对照稿（每段原文+修改后）
- 纯文本（供检测用）
- 保存到 `E:\Obsidian_001\仓库001\`

### Step 5: Self-check
Run rule-based feature scan to verify improvement:
- Count frequency of AI marker words (本文/目前/随着/因此)
- Check sentence length variance (higher = more human-like)
- Check paragraph opening diversity
- Report before/after comparison

### Step 6: Report to User
Tell the user:
1. What style was generated
2. Key metrics (AI features removed, changes made)
3. **Ask them to verify with their school's tool** (知网AIGC/PaperYY/维普)
4. If they send back flagged paragraphs, do second-round targeted cleanup

## Required Tools
- python-docx: `pip install python-docx`
- Python 3.8+

## Output Convention
Default output directory: `E:\Obsidian_001\仓库001\`
File naming: `《论文名》降AI修改稿_风格.md` + `《论文名》降AI纯文本_风格.txt`
