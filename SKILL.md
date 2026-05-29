---
name: doc2deck
description: "Use whenever the user wants to convert a Word document (.docx) into a polished PowerPoint presentation (.pptx). This skill specializes in academic reports, papers, and theses — transforming lengthy Chinese documents into structured, visually modern slide decks suitable for defense or presentation. Also use when the user needs to proofread a Chinese academic .docx, create a tracked-changes revision, or generate slides from any structured document. Triggers: 'convert docx to pptx', 'make slides from my report', 'turn my paper into a presentation', 'create a deck', 'generate PPT from Word', 'docx转pptx', '生成PPT', '做 slides', '简报', '论文答辩PPT', or any request involving .docx → .pptx conversion. Also trigger when the user has a written report and needs a presentation from it, or when they ask to proofread and format a Chinese academic document. Optimized for Chinese academic content with mixed Chinese-English formatting."
license: MIT
---

# Doc2Deck (简报智造)

Convert academic .docx reports into professional presentation decks through a pipeline of: proofreading → intelligent restructuring → interactive review → code-driven PPTX generation.

**Target audience:** University students preparing thesis defenses, project reports, or academic presentations.

**Primary language:** Chinese (Simplified), with robust handling of mixed Chinese-English academic content (English terminology, references, figures).

## Quick Reference

| Task | Approach |
|------|----------|
| Read/extract .docx content | `pandoc input.docx -t markdown --wrap=none -o content.md` |
| Read/extract .pptx content | `python -m markitdown presentation.pptx` |
| Proofread with tracked changes | Unpack → edit XML → repack (see Phase 1) |
| Create .pptx from scratch | pptxgenjs (see [pptxgenjs.md](pptxgenjs.md) for full API) |
| Edit existing .pptx from template | Unpack → edit slides → clean → repack (see [editing.md](editing.md)) |
| Visual slide overview | `python scripts/thumbnail.py presentation.pptx` |
| Visual QA (high-res) | `soffice` → `pdftoppm` (see Phase 4) |
| Accept all tracked changes | `python scripts/accept_changes.py input.docx output.docx` |

## Scripts Reference

| Script | Purpose |
|--------|---------|
| `scripts/office/unpack.py` | Extract OOXML, pretty-print, merge runs, escape smart quotes |
| `scripts/office/pack.py` | Validate, auto-repair, condense XML, repack to OOXML |
| `scripts/office/validate.py` | OOXML schema validation |
| `scripts/office/soffice.py` | LibreOffice wrapper (auto-configures sandboxed paths) |
| `scripts/accept_changes.py` | Accept all tracked changes (requires LibreOffice) |
| `scripts/comment.py` | Add comment boilerplate across multiple XML files |
| `scripts/add_slide.py` | Duplicate slide or create from layout |
| `scripts/clean.py` | Remove orphaned slides, media, and relationships |
| `scripts/thumbnail.py` | Create visual grid of slides (for template analysis) |

---

## Workflow

Doc2Deck runs in four sequential phases. Never skip a phase or proceed without completing the previous one.

---

## Phase 1: Ingestion & Proofreading

### 1.1 Read the Document

Extract the full text content from the .docx file:

```bash
pandoc input.docx -t markdown --wrap=none -o content.md
```

Alternatively, for more detail on formatting and styles:

```bash
pandoc input.docx -t plain --wrap=none -o content.txt
```

Read the extracted content thoroughly. Identify:
- **Heading hierarchy** (H1 → slide title, H2 → section header, H3 → sub-point)
- **Body paragraphs** and their length
- **Bold/emphasized text** (key findings, numbers, terms)
- **Image placeholders** (figure captions, "图X", "表X" references)
- **Tables** and their structure

### 1.2 Proofread: Automatic vs. Manual Boundary

Proofreading is divided into two categories with a hard boundary. **Never cross this boundary without user approval.**

#### 1.2.1 Automatic Corrections (No User Confirmation Needed)

These are surface-level formatting and grammar errors. Apply them directly — do not ask the user.

**Punctuation (highest priority):**
- Mixed Chinese/English marks: `，。！？` (CN) vs `,.!?` (EN). Chinese text must use Chinese punctuation.
- Incorrect: `实验结果表明,该方法的准确率为95%.`
- Correct: `实验结果表明，该方法的准确率为95%。`
- Parentheses in Chinese context: use full-width `（）` for Chinese, half-width `()` for English content only.

**English in Chinese text:**
- English words/terms need a space before and after when sandwiched between Chinese characters.
- Incorrect: `使用CNN模型进行训练`
- Correct: `使用 CNN 模型进行训练`
- Exception: no space needed when followed by Chinese punctuation: `使用 CNN。`

**Common Chinese typos:**
- 的/地/得 confusion
- 做/作 confusion
- 在/再 confusion
- 其他/其它 consistency

**Formatting consistency:**
- Check heading numbering: ensure sequential (一、二、三... or 1, 2, 3...)
- Paragraph first-line indent: all body paragraphs should use consistent indentation
- Figure/table numbering: check for gaps or duplicates

All automatic corrections go directly into the tracked-changes document (see 1.3).

#### 1.2.2 Content-Level Suggestions (User Confirmation REQUIRED)

These go beyond surface formatting — they affect meaning, structure, or argumentation. **You MUST present these to the user and wait for explicit approval before making any changes.**

**What falls into this category:**
- **Ambiguous or unclear sentences** — flag and suggest rewording, but do not rewrite without approval
- **Missing information** — e.g., "此处缺少样本量说明，建议补充" or "未交代实验环境配置"
- **Structural issues** — e.g., "第3章与第4章之间缺少过渡段落，建议增加承上启下的内容"
- **Data presentation** — e.g., "表2的数据对比建议补充统计显著性（p值）"
- **Terminology inconsistency** — e.g., 同一概念在全文中混用 "机器学习" / "深度学习" / "AI"
- **Logical gaps** — e.g., "结论部分提及了方法A优于方法B，但在实验部分未见直接对比"

**How to present:**
```
## 内容建议（需确认）

以下为可能影响内容表达的建议，请逐条确认是否采纳：

1. **[位置/章节]** [具体问题和建议]
   - 原文：...
   - 建议：...
   
2. **[位置/章节]** [具体问题和建议]
   - 原文：...
   - 建议：...
```

**User responses:**
- "全部采纳" / "采纳1,3,5" / "采纳第1条和第4条" → apply approved changes
- "都不需要" → skip all content changes
- "第2条改成..." → apply user's alternative version

**After user approves, apply the confirmed changes to the tracked-changes document alongside the automatic corrections.**

**Never rewrite content autonomously.** If you are unsure whether something is format or content, present it to the user.

### 1.3 Generate Tracked-Changes Document

After identifying all corrections, produce a proofread .docx with corrections tracked. This uses the docx skill's XML tracked-changes workflow directly.

**Step 1: Unpack the original document:**
```bash
python scripts/office/unpack.py input.docx unpacked/
```

**Step 2: Edit the XML** to add `<w:del>` / `<w:ins>` for each correction:

```xml
<!-- Insertion -->
<w:ins w:id="1" w:author="Doc2Deck" w:date="2026-01-01T00:00:00Z">
  <w:r><w:rPr><!-- copy original rPr for formatting --></w:rPr><w:t>inserted text</w:t></w:r>
</w:ins>

<!-- Deletion -->
<w:del w:id="2" w:author="Doc2Deck" w:date="2026-01-01T00:00:00Z">
  <w:r><w:rPr><!-- copy original rPr --></w:rPr><w:delText>deleted text</w:delText></w:r>
</w:del>
```

**Critical XML rules for tracked changes:**
- Replace entire `<w:r>` blocks — nest `<w:del>`/`<w:ins>` as siblings of runs, never inside them
- Preserve `<w:rPr>` formatting from the original run
- Use minimal edits — only mark what changes, leave surrounding text untouched
- Inside `<w:del>`: use `<w:delText>` instead of `<w:t>`, and `<w:delInstrText>` instead of `<w:instrText>`
- Deleting ALL content from a paragraph: also add `<w:del/>` inside `<w:pPr><w:rPr>` to mark the paragraph mark as deleted
- Rejecting another author's insertion: nest `<w:del>` inside their `<w:ins>`
- Restoring another author's deletion: add `<w:ins>` after their `<w:del>` (don't modify their deletion)
- Use "Doc2Deck" as the author
- Each tracked change needs a unique `w:id`

**Adding comments:**
```bash
python scripts/comment.py unpacked/ 0 "Comment text with &amp; and &#x2019;" --author "Doc2Deck"
python scripts/comment.py unpacked/ 1 "Reply text" --parent 0 --author "Doc2Deck"
```

Then add comment markers in document.xml — `<w:commentRangeStart>` and `<w:commentRangeEnd>` are **siblings** of `<w:r>`, never inside a run.

**Step 3: Repack the proofread document:**
```bash
python scripts/office/pack.py unpacked/ proofread.docx --original input.docx
```

**Auto-repair during pack:**
- `durableId` >= 0x7FFFFFFF → regenerates valid ID
- Missing `xml:space="preserve"` on `<w:t>` with whitespace → adds it

Present a proofreading summary:

"自动修正已完成，共 X 处（Y 处标点修正，Z 处错别字，W 处格式统一）。修订版已保存至 proofread.docx。"

Then immediately present content-level suggestions (see 1.2.2) for user review. Do NOT proceed to Phase 2 until all content suggestions are resolved.

---

## Phase 2: Restructuring & Two-Step Review

### 2.1 Generate Slide Outline

Based on the proofread content, create a structured slide-by-slide outline. Follow academic presentation logic:

| Slide | Type | Content |
|-------|------|---------|
| 1 | Title | Report title, author, date, advisor |
| 2 | Outline | Presentation structure overview (3-5 items) |
| 3-4 | Background | Research context, problem statement, literature gaps |
| 5-7 | Methods | Approach, experimental design, key techniques |
| 8-10 | Results | Key findings, data highlights, comparisons |
| 11 | Discussion | Analysis, limitations, significance |
| 12 | Conclusion | Summary, contributions, future work |
| 13 | Thanks | Acknowledgments, Q&A |

**Rules for outline construction:**
- Each content slide must have 3-5 bullet points maximum
- One key idea per slide — if a section has more than 5 points, split it into multiple slides
- Identify where visuals are needed: "此处建议插入[图表类型]展示[内容]"
- Mark slides where data can be turned into charts

### 2.2 Step 1: Outline Review (First Intercept)

Present the outline to the user in this format:

```
## 幻灯片结构大纲

**共 N 页**

1. 标题页：[标题]
2. 目录：[5个章节标题]
3. 背景-1：[研究背景]
...

## 审稿建议

- 第X页：[具体建议，如"建议补充算法参数"]
- 第Y页：[具体建议，如"此处数据对比适合用柱状图呈现"]
```

**Actively provide substantive suggestions.** Look for:
- Missing methodological details (algorithms, parameters, sample sizes)
- Data that would benefit from visualization
- Logical gaps or transitions that need smoothing
- Sections where the user's expertise could add depth

Wait for user confirmation. The user may:
- Approve as-is: "确认" / "OK" / "没问题"
- Modify the outline: "第3页和第4页合并" / "在方法部分加一页讲数据集"
- Add suggestions to incorporate: any feedback they provide

If the user requests changes, update the outline and re-present. Do not proceed until the user confirms the outline.

### 2.3 Generate Full Markdown Draft

Once the outline is confirmed, expand each slide into its full content. Write a complete Markdown document where:

- Each `## Slide N: Title` block represents one slide
- Slide content uses bullet lists (3-5 items per slide)
- Data callouts use `> **关键数据：** ...` blockquotes
- Placeholder hints use `<!-- 建议：... -->` HTML comments
- Chart suggestions use `<!-- CHART: type, data description -->`

**Condensation rules:**
- Turn paragraphs into scannable bullet points
- Keep each bullet to 1-2 lines
- Extract numbers and key terms; discard verbose exposition
- Preserve English technical terms as-is (e.g., "ResNet-50", "p < 0.01")

### 2.4 Step 2: Draft Review (Second Intercept)

Present the full Markdown draft to the user. Include a second round of suggestions focused on slide-level polish:

- "第X页要点过多（N条），建议精简至3-5条"
- "第Y页标题与内容不够匹配，建议改为..."
- "第Z页缺少过渡句，听众可能跟不上逻辑"

Wait for user confirmation. The user may edit the Markdown directly or provide verbal feedback. Iterate until confirmed.

---

## Phase 3: PPTX Generation

### 3.1 Design System

Use a fixed academic presentation style. Read [references/academic-style.md](references/academic-style.md) for the complete design system including color palettes, typography, and all six slide templates with implementation code.

**Core palette (Academic Navy):**
- Primary: `1E2761` (deep navy) — title slides, section headers, accent shapes
- Secondary: `CADCFC` (ice blue) — subtle backgrounds, card fills
- Accent: `F4A261` (warm gold) — data highlights, key numbers
- Text: `1A1A1A` (near-black) on light backgrounds, `FFFFFF` (white) on dark backgrounds
- Background: `FAFAFA` (off-white) for content slides, dark navy for title/conclusion

**Typography:** 微软雅黑 (Microsoft YaHei) for all text — universally available on Chinese university computers.

### 3.2 Generate Slides with pptxgenjs

Create slides programmatically using pptxgenjs. Read **[pptxgenjs.md](pptxgenjs.md)** for the complete API reference (text formatting, shapes, images, tables, charts, icons). Below are the doc2deck-specific slide implementations.

**Setup:**
```javascript
const pptxgen = require("pptxgenjs");
const pres = new pptxgen();
pres.layout = "LAYOUT_16x9";  // 10" × 5.625"
pres.author = "Doc2Deck";
pres.title = "[Presentation Title]";
```

**Slide types to implement (see [academic-style.md](references/academic-style.md) for pixel-level specs):**

**Title slide:** Full `1E2761` background, white title 40pt bold centered, `F4A261` gold accent line, subtitle 18pt in `CADCFC`, author/date in `CADCFC` 14pt.

**Content slides:** `FAFAFA` background, left-aligned title 32pt bold in `1E2761`, body text 16pt in `1A1A1A`, `F4A261` thin underline beneath title. Bullet points with `bullet: true`. Navy footer bar with white page number 9pt.

**Section divider slides:** Full `1E2761` background, section number 60pt `F4A261` bold, section title 36pt `FFFFFF` bold, thin gold accent line, subtitle in `CADCFC` 16pt.

**Data callout slides:** White card-shaped `RECTANGLE` with subtle shadow, stat number 40pt `F4A261` bold, label 14pt `6B7280`. Three cards per row.

**Two-column comparison:** Side-by-side white card shapes with headers, pros/cons, and use-case sections.

**Conclusion slide:** Same structure as title slide but with key takeaways.

### 3.3 PptxGenJS Critical Rules

These issues cause file corruption, visual bugs, or broken output. Always follow:

- **NEVER use `#` with hex colors** — `color: "FF0000"` ✅, `color: "#FF0000"` ❌
- **NEVER encode opacity in 8-char hex** — use the `opacity` property instead
- **Use `bullet: true`**, never unicode `•` symbols (creates double bullets)
- **Use `breakLine: true`** between array items or text runs together
- **NEVER reuse option objects** — PptxGenJS mutates objects in-place. Use factory functions.
- **Each presentation needs a fresh `pptxgen()` instance**
- **Shadow `offset` must be non-negative** — use `angle` to control direction
- **`ROUNDED_RECTANGLE` + rectangular accent overlays don't mix** — use `RECTANGLE` instead
- **Set `margin: 0`** on text boxes when aligning with shapes/icons at same x-position

See [pptxgenjs.md](pptxgenjs.md) for the full list of pitfalls and examples.

### 3.4 Text Overflow Prevention

Hard limits to prevent overflow (Chinese text at 16pt in 微软雅黑: ~33 chars/line in 8.6" wide text area):

| Element | Max Items | Max Chars Each |
|---------|-----------|----------------|
| Content slide title | 1 | 25 |
| Top-level bullet | 5 | 50 |
| Sub-bullet | 3 per parent | 60 |
| Stat card label | 1 | 10 |

If content exceeds these limits, add a new slide rather than shrinking fonts.

### 3.5 Generate the File

```javascript
pres.writeFile({ fileName: "presentation.pptx" });
```

---

## Phase 4: Quality Assurance

Follow the pptx skill's QA process. **Assume there are problems. Your job is to find them.** First render is almost never correct — approach QA as a bug hunt, not a confirmation step.

### 4.1 Content QA

```bash
python -m markitdown presentation.pptx
```

Check for missing content, typos, wrong order, placeholder text.

```bash
python -m markitdown presentation.pptx | grep -iE "xxxx|lorem|ipsum|placeholder"
```

If grep returns results, fix them before declaring success.

### 4.2 Visual QA

**⚠️ USE SUBAGENTS** — even for 2-3 slides. Fresh eyes catch what code-blinded eyes miss.

Convert slides to high-resolution images:

```bash
python scripts/office/soffice.py --headless --convert-to pdf presentation.pptx
pdftoppm -jpeg -r 150 presentation.pdf slide
```

To re-render specific slides after fixes:

```bash
pdftoppm -jpeg -r 150 -f N -l N presentation.pdf slide-fixed
```

Then use this prompt with subagents:

> Visually inspect these slides. Assume there are issues — find them.
>
> Look for:
> - Overlapping elements (text through shapes, lines through words)
> - Text overflow or cut off at box boundaries
> - Uneven gaps between content blocks
> - Insufficient margin from slide edges (< 0.5")
> - Low-contrast text or icons
> - Text boxes too narrow causing excessive wrapping
> - Leftover placeholder content
>
> Report ALL issues found, including minor ones.

### 4.3 Verification Loop

1. Generate slides → Convert to images → Inspect
2. **List issues found** (if zero, look again more critically)
3. Fix issues
4. **Re-verify affected slides** — one fix often creates another problem
5. Repeat until a full pass reveals no new issues

**Do not declare success until you've completed at least one fix-and-verify cycle.**

---

## Working with Existing PPTX Templates

**When to use this path:** Only when the user explicitly provides a `.pptx` file to use as a template (e.g., "use this template", school/organization branded template, or a reference presentation to match). In all other cases, use the main pipeline (Phases 1-4) with the fixed Academic Navy design system.

When a reference .pptx is provided, follow the template-based workflow in [editing.md](editing.md). The template's existing design (colors, layouts, fonts) takes precedence over the Academic Navy style — adapt content to the template, not vice versa.

1. Analyze template: `python scripts/thumbnail.py template.pptx` + `python -m markitdown template.pptx`
2. Plan slide mapping — match content to varied layouts (avoid repeating the same layout)
3. Unpack: `python scripts/office/unpack.py template.pptx unpacked/`
4. Build structure: delete/slide/reorder slides in `<p:sldIdLst>`, complete all structural changes first
5. Edit content: use subagents for parallel slide editing (each slide is a separate XML file)
6. Clean: `python scripts/clean.py unpacked/`
7. Pack: `python scripts/office/pack.py unpacked/ output.pptx --original template.pptx`

See [editing.md](editing.md) for formatting rules, common pitfalls, and XML examples.

---

## Converting .doc to .docx

Legacy `.doc` files must be converted before processing:

```bash
python scripts/office/soffice.py --headless --convert-to docx document.doc
```

---

## Dependencies

- **pandoc**: Text extraction from .docx
- **pptxgenjs**: `npm install -g pptxgenjs` — create .pptx from scratch
- **markitdown**: `pip install "markitdown[pptx]"` — text extraction from .pptx
- **LibreOffice** (`soffice`): PDF conversion and .doc→.docx conversion (auto-configured via `scripts/office/soffice.py`)
- **Poppler** (`pdftoppm`): PDF to slide images for visual QA
- **Pillow**: `pip install Pillow` — thumbnail grid generation for template analysis
- **react-icons + sharp**: `npm install -g react-icons react react-dom sharp` — SVG icons rasterized to PNG (optional, for icon usage in slides)

---

## Key Principles

- **Automatic vs. manual boundary is absolute.** Format/grammar fixes are auto-applied. Content changes (rewording, adding info, restructuring arguments) MUST be approved by the user first. Never cross this line.
- **Never skip the review phases.** The two-step confirmation gives the presentation the user's unique perspective and depth.
- **Academic integrity over visual flashiness.** Subtle, professional design beats loud decoration for defense presentations.
- **Chinese text needs more breathing room.** More line spacing and shorter line lengths than English.
- **Numbers tell stories.** When you see data in the report (percentages, comparisons, trends), proactively suggest chart-based slides.
- **One idea per slide.** If you're tempted to put 7+ bullets on a slide, split it.
- **First render is always wrong.** Approach QA skeptically — find problems, fix, and re-verify.
- **Use subagents for parallel work.** Slide-by-slide editing and visual QA both benefit from parallel subagents with fresh eyes.
