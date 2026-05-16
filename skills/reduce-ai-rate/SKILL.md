---
name: reduce-ai-rate
description: Use when a user provides a Chinese academic paper (.docx) with high AI detection rate and needs to lower it below a target threshold (e.g. <25%). The user explicitly mentions "AI率" or "降AI" or "AIGC检测" or their university flagged the paper for AI-generated content.
---

# Reduce AI Rate (论文降AI率)

## Overview
Systematic workflow for reducing AI detection rate in Chinese academic papers. Reads a .docx file, classifies paragraphs, applies targeted rewriting based on user's style preference, and outputs a clean version. The core insight: **commercial AI detectors (知网/维普/PaperYY)抓的不是"AI概率"，而是"AI写作特征"——句长均匀、排比密集、连接词模式化、缺乏个人观点。消除这些特征比拟合任何检测模型更有效。**

## Workflow

```
用户输入.docx路径
  → Step 0: 收集信息(风格1/2/3 + 降AI目标阈值 + 输出文件选择 + 保留段落)
  → python-docx提取纯文本 + 保留原始格式结构用于.docx生成
  → 分类段落: keep(封面/声明/目录/图题/参考文献/用户指定段落) vs rewrite(正文)
  → 按所选风格的策略改写
  → 输出用户选定的文件类型(Markdown对照稿/纯文本/新.docx)到目标目录
  → 自检(对比阈值 → 未达标则迭代改写)
  → 告诉用户结果 + 询问是否删除skill
```

## Style Options

Present the three options as a numbered menu:

```
请选择改写风格（输入 1/2/3）：
1. 专业版 — 替换高频词 + 打破排比结构 + 句首多样化 + 拉大句长方差，保留学术术语和正式语气
   → 预期降AI率约 15% | 学术感 ★★★★★
2. 平衡版 — 在专业版基础上 + 适度个人视角 + 自然过渡词
   → 预期降AI率约 20% | 学术感 ★★★★
3. 口语化版 — 在平衡版基础上 + 大量口语表达 + 打散段落结构
   → 预期降AI率约 30% | 学术感 ★★★
```

注意：以上数值基于v3模型。不同学校工具数值不同，但**相对比例一致**——口语化版一定比专业版降得多。

## Detailed Workflow

### Step 0: 用户信息收集

按顺序逐项询问用户（不要一次性全问）：

**0.1 论文路径**
```
请输入 .docx 文件路径：
```
用户输入后验证文件存在。

**0.2 风格选择**
显示编号菜单（见上方 Style Options），用户输入 1/2/3。

**0.3 目标降AI率阈值**
```
请输入目标降AI率阈值（如输入 20 表示降至 20% 以下）：
```
用户输入数值。后续自检时对比此阈值，未达标则迭代改写。

**0.4 输出文件选择**
逐项询问（默认 y）：
```
是否生成 .docx 修改稿？（y/n，默认 y）
是否生成 Markdown 对照稿？（y/n，默认 y）
是否生成 纯文本用于检测？（y/n，默认 y）
```

**0.5 保留段落**
```
有哪些段落/章节需要完全跳过、不做任何修改？
（如：第三章、参考文献格式、所有公式章节。没有则直接回车）
```
用户指定的部分叠加在 Step 2 的 keep 列表之上，连词汇替换都不做。

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
- **用户指定的保留段落**（来自 Step 0.5，完全跳过，不做任何修改）

Rewrite: Everything else (摘要、引言、正文各章节、总结、致谢)

### Step 3: Apply Rewriting Based on Style

**专业版手法：**

词汇替换（必要但不充分——仅做这层不够）：
- "本文" → "本研究"/"此次设计"
- "目前" → "现阶段"/去掉
- "随着" → 直接陈述
- "因此" → "基于此"/"鉴于此"

句式结构改写（关键核心——真正降低AI感的手段）：
- **排比句拆解**：不保留"首先/其次/最后""第一/第二/第三"结构，拆成2-3句，每句换主语或换角度
  - ❌ "首先分析了...其次讨论了...最后总结了..."
  - ✅ "研究从...入手。在这个基础上，进一步考察了...相关的问题。由此得出的核心认识是..."
- **段落首句多样化**：不都用"XX是YY的重要ZZ"模板开场
  - ❌ "图像分割是计算机视觉的重要任务。目前主流方法包括..."
  - ✅ "做图像分割的时候，第一个要解决的问题就是..."
  - ✅ "从...入手，是处理...问题的一个常见思路。"
  - （"做...的时候""入手"不是口语化，而是中文自然写作的常见开篇）
- **句长方差拉大**：连续长句之间插入简短评论句
  - 短句如："这一点值得注意。" / "但情况并非总是如此。" / "这里存在一个明显的矛盾。"
- **逻辑衔接词替换**：用"学术但非模板化"的表达
  - "因此" → "基于上述分析"/"由此出发"/"这就意味着"
  - "综上所述" → "综合以上讨论"/"将上述几方面结合起来看"
  - "例如" → "以...为例"/"这里不妨看一下"/"具体来说"
- **保留**：所有专业术语、学术深度、正式语气

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
Only generate the file types the user selected in Step 0.4（默认全生成）:
- 如果选了 .docx → 按下方方法生成
- 如果选了 Markdown 对照稿 → 生成
- 如果选了 纯文本 → 生成

保存位置：默认与输入文件同目录，用户可指定自定义路径。

#### .docx 生成方法（仅在用户选择时才执行）
核心原则：**打开原始文档副本，原位替换文本，不动非文本元素（图片、形状、公式）。**

```python
from docx import Document
import os

# 输出目录：用户指定则用自定义路径，否则与输入文件同目录
if user_provided_output_dir:
    output_dir = user_provided_output_dir
else:
    output_dir = os.path.dirname(original_path)
os.makedirs(output_dir, exist_ok=True)

def has_image(run):
    """检查 run 是否包含图片或非文本元素"""
    ns = {
        'w': 'http://schemas.openxmlformats.org/wordprocessingml/2006/main',
        'wp': 'http://schemas.openxmlformats.org/drawingml/2006/wordprocessingDrawing',
        'mc': 'http://schemas.openxmlformats.org/markup-compatibility/2006',
    }
    r = run._element
    if r.find('.//{http://schemas.openxmlformats.org/wordprocessingml/2006/main}drawing', ns) is not None:
        return True
    if r.find('.//{http://schemas.openxmlformats.org/wordprocessingml/2006/main}pict', ns) is not None:
        return True
    return False

def get_text_run_format(para):
    """取第一个非空文本 run 的格式"""
    for r in para.runs:
        if r.text.strip() and not has_image(r):
            return {
                'name': r.font.name,
                'size': r.font.size,
                'bold': r.font.bold,
                'italic': r.font.italic,
                'color': r.font.color.rgb if r.font.color and r.font.color.rgb else None,
            }
    return {}

# 1. 打开原文档
doc = Document(original_path)

# 2. 将改写文本按段落索引映射（与提取纯文本时的顺序一致）
rewritten_map = {idx: text for idx, text in enumerate(all_rewritten_texts)}

# 3. 遍历段落，原位替换
for i, para in enumerate(doc.paragraphs):
    if i not in rewritten_map:
        continue
    new_text = rewritten_map[i]
    if new_text == para.text:
        continue
    
    # 获取格式信息
    fmt = get_text_run_format(para)
    
    if any(has_image(r) for r in para.runs):
        # 含图片段落：保留图片 run，只清空文本 run
        for r in para.runs:
            if not has_image(r):
                r.text = ''
        new_run = para.add_run(new_text)
    else:
        # 无图片段落：整体替换
        for r in list(para.runs):
            r._element.getparent().remove(r._element)
        new_run = para.add_run(new_text)
    
    # 应用原格式
    if fmt.get('name'):
        new_run.font.name = fmt['name']
    if fmt.get('size'):
        new_run.font.size = fmt['size']
    if fmt.get('bold'):
        new_run.font.bold = fmt['bold']
    if fmt.get('color'):
        new_run.font.color.rgb = fmt['color']

# 4. 处理表格中的段落
for table in doc.tables:
    for row in table.rows:
        for cell in row.cells:
            for para in cell.paragraphs:
                if para.text.strip():
                    # 表格文本做简单关键词替换，不做结构改写
                    new_cell_text = para.text
                    for old_word, new_word in simple_replace_rules.items():
                        new_cell_text = new_cell_text.replace(old_word, new_word)
                    if new_cell_text != para.text:
                        for r in list(para.runs):
                            r._element.getparent().remove(r._element)
                        para.add_run(new_cell_text)

# 5. 保存
output_path = os.path.join(output_dir, f'《{paper_name}》降AI修改稿_{style}.docx')
doc.save(output_path)
```

#### 图片处理说明
- 包含图片（drawing/pict 元素）的 run **不会被删除或修改**
- 图片在原段落内的相对位置不变
- ⚠️ 文字改写后的长度变化会导致 Word 重新分页，图片可能出现在不同页面——**这无法避免，属正常现象**。告知用户用 Word 打开后微调分页
- 文件保存后建议用 Word"打印预览"或"阅读视图"检查排版

### Step 5: Self-check & Iteration
Run rule-based feature scan:
- Count frequency of AI marker words (本文/目前/随着/因此)
- Check sentence length variance (higher = more human-like)
- Check paragraph opening diversity
- Report before/after comparison
- **估计当前降AI率**（基于扫描结果的估算）

**与 Step 0.3 的阈值对比：**
- 如果估算值 **未达到** 用户设定的阈值 → 自动回到 Step 3，对仍存在 AI 特征的段落进行**第二轮改写**。再用更激进的手法处理一遍，然后重新自检。
- 如果估算值 **已达到或低于** 阈值 → 进入 Step 6。
- 如果经过 3 轮迭代仍未达标 → 进入 Step 6 并告知用户当前结果，建议用学校工具终检。

### Step 6: Report & Post-processing
Tell the user:
1. What style was generated
2. Key metrics (AI features removed, changes made)
3. **Ask them to verify with their school's tool** (知网AIGC/PaperYY/维普)
4. If they send back flagged paragraphs, do second-round targeted cleanup
5. **询问是否删除 skill**:
   ```
   是否删除 reduce-ai-rate skill？（y/n）
   ```
   - 如果 y → 删除 `~/.claude/skills/reduce-ai-rate/` 目录
   - 如果 n → 保持不动

## Required Tools
- python-docx: `pip install python-docx`
- Python 3.8+

## Output Convention
Default output directory: same directory as the input .docx file (用户可指定自定义路径)
File naming:
- `《论文名》降AI修改稿_风格.md` — Markdown 对照稿
- `《论文名》降AI纯文本_风格.txt` — 纯文本（供检测）
- `《论文名》降AI修改稿_风格.docx` — 修改后的 .docx（保留原始格式）
