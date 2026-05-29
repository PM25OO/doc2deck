# Academic Presentation Style Guide

Design system for Doc2Deck. Fixed academic style optimized for Chinese university defense presentations.

---

## Color Palette

### Primary: Academic Navy

| Role | Hex | Usage |
|------|-----|-------|
| Primary | `1E2761` | Title slides, section headers, accent shapes, footer bars |
| Secondary | `CADCFC` | Card backgrounds, subtle fills, chart grid lines |
| Accent | `F4A261` | Key numbers, data highlights, important callouts |
| Text Dark | `1A1A1A` | Body text on light backgrounds |
| Text Light | `FFFFFF` | Text on dark backgrounds |
| Muted | `6B7280` | Captions, footnotes, secondary info |
| Background Light | `FAFAFA` | Content slide backgrounds |
| Background Dark | `1E2761` | Title/section divider/conclusion backgrounds |

### When to Use Each Color

- **Title slide:** Full `1E2761` background. Title in `FFFFFF` 40pt bold. Subtitle in `CADCFC` 18pt.
- **Content slides:** `FAFAFA` background. Title in `1E2761` 32pt bold. Body in `1A1A1A` 16pt. Bullet markers in `1E2761`.
- **Section dividers:** Full `1E2761` background. Section number in `F4A261` 14pt. Section title in `FFFFFF` 36pt.
- **Data highlights:** Number in `F4A261` 60pt bold. Label below in `1A1A1A` 14pt.
- **Footer bar:** `1E2761` thin bar at slide bottom, page number in `FFFFFF` 9pt.

---

## Typography

### Font Stack

For Chinese presentations, system fonts that work across platforms:

| Platform | Header Font | Body Font |
|----------|-------------|-----------|
| Windows | 微软雅黑 (Microsoft YaHei) | 微软雅黑 (Microsoft YaHei) |
| macOS | PingFang SC | PingFang SC |
| Fallback | Arial (for English text within) | Arial |

Use 微软雅黑 as the primary font — it's universally available on Chinese university computers and reads well on projectors.

### Size Hierarchy

| Element | Size | Weight | Color |
|---------|------|--------|-------|
| Title slide title | 40pt | Bold | `FFFFFF` |
| Title slide subtitle | 18pt | Regular | `CADCFC` |
| Content slide title | 32pt | Bold | `1E2761` |
| Section header | 36pt | Bold | `FFFFFF` |
| Body text | 16pt | Regular | `1A1A1A` |
| Bullet sub-text | 14pt | Regular | `4A4A4A` |
| Data callout number | 60pt | Bold | `F4A261` |
| Data callout label | 14pt | Regular | `6B7280` |
| Footer/page number | 9pt | Regular | `FFFFFF` |
| Chart labels | 10pt | Regular | `6B7280` |

---

## Layout Specifications

### Slide Dimensions

16:9 widescreen (10" × 5.625") — standard for modern projectors.

### Margins

- **Content slides:** 0.7" left, 0.7" right, 0.6" top (below title), 0.5" bottom (above footer)
- **Title slides:** content centered vertically and horizontally
- **Section dividers:** content centered

### Content Area

- Usable width: 8.6 inches (10" minus 1.4" margins)
- Title zone: top 1.0 inch of slide
- Body zone: 1.0" to 4.8" from top
- Footer zone: bottom 0.4" (page number bar)

### Spacing

- Between bullet items: 0.3"
- Between sections within a slide: 0.5"
- Between title and body: 0.3"
- Card padding: 0.15" internal

---

## Slide Templates

### 1. Title Slide

```
┌──────────────────────────────────┐
│ [navy background 1E2761]         │
│                                  │
│                                  │
│   报告标题 (40pt bold white)      │
│   ── thin gold line ──           │
│   副标题/作者 (18pt ice blue)     │
│                                  │
│                                  │
│   姓名 | 导师 | 日期 (14pt muted) │
│                                  │
└──────────────────────────────────┘
```

Implementation:
```javascript
slide.background = { color: "1E2761" };
// Title centered, y: 1.8
slide.addText("报告标题", { x: 0.5, y: 1.8, w: 9, h: 0.9, fontSize: 40, fontFace: "Microsoft YaHei", color: "FFFFFF", bold: true, align: "center" });
// Gold accent line
slide.addShape(pres.shapes.RECTANGLE, { x: 3.5, y: 2.8, w: 3, h: 0.03, fill: { color: "F4A261" } });
// Subtitle
slide.addText("副标题", { x: 0.5, y: 3.1, w: 9, h: 0.6, fontSize: 18, fontFace: "Microsoft YaHei", color: "CADCFC", align: "center" });
// Author info
slide.addText("姓名 | 导师 | 2026", { x: 0.5, y: 4.5, w: 9, h: 0.4, fontSize: 14, fontFace: "Microsoft YaHei", color: "CADCFC", align: "center" });
```

### 2. Content Slide (Standard Bullet)

```
┌──────────────────────────────────┐
│ 页面标题 (32pt bold navy)         │
│                                  │
│  ● 要点一 (16pt, dark)           │
│  ● 要点二                        │
│    └ 子要点 (14pt muted)         │
│  ● 要点三                        │
│  ● 要点四                        │
│                                  │
│ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓  │ ← navy bar
│ 页面 3/12                        │
└──────────────────────────────────┘
```

Implementation:
```javascript
slide.background = { color: "FAFAFA" };
// Title
slide.addText("页面标题", { x: 0.7, y: 0.3, w: 8.6, h: 0.7, fontSize: 32, fontFace: "Microsoft YaHei", color: "1E2761", bold: true, margin: 0 });
// Thin underline below title
slide.addShape(pres.shapes.RECTANGLE, { x: 0.7, y: 0.95, w: 2, h: 0.04, fill: { color: "F4A261" } });
// Bullet points
slide.addText(bulletItems, { x: 0.9, y: 1.3, w: 8.2, h: 3.5, fontSize: 16, fontFace: "Microsoft YaHei", color: "1A1A1A" });
// Footer bar
slide.addShape(pres.shapes.RECTANGLE, { x: 0, y: 5.2, w: 10, h: 0.425, fill: { color: "1E2761" } });
slide.addText("3 / 12", { x: 0.5, y: 5.2, w: 1, h: 0.425, fontSize: 9, fontFace: "Microsoft YaHei", color: "FFFFFF", align: "center" });
```

### 3. Section Divider

```
┌──────────────────────────────────┐
│ [navy background 1E2761]         │
│                                  │
│       03 (60pt gold bold)        │
│   研究方法 (36pt white bold)      │
│   ── thin gold line ──           │
│   实验设计与技术路线 (16pt ice blue) │
│                                  │
└──────────────────────────────────┘
```

### 4. Data Callout Slide

```
┌──────────────────────────────────┐
│ 关键成果 (32pt bold navy)         │
│                                  │
│   95.2%        3.2×         12项 │
│   准确率        速度提升      创新点 │
│                                  │
│  [chart area: 6" × 3"]          │
│                                  │
└──────────────────────────────────┘
```

Data callout implementation — three stat cards with gold numbers:
```javascript
// Three stat boxes in a row
const stats = [
  { num: "95.2%", label: "准确率" },
  { num: "3.2×", label: "速度提升" },
  { num: "12项", label: "创新点" },
];
stats.forEach((s, i) => {
  const x = 0.7 + i * 3.0;
  slide.addShape(pres.shapes.RECTANGLE, { x, y: 1.5, w: 2.6, h: 1.8, fill: { color: "FFFFFF" },
    shadow: { type: "outer", color: "000000", blur: 4, offset: 2, angle: 135, opacity: 0.08 } });
  slide.addText(s.num, { x, y: 1.6, w: 2.6, h: 0.9, fontSize: 40, fontFace: "Microsoft YaHei", color: "F4A261", bold: true, align: "center" });
  slide.addText(s.label, { x, y: 2.5, w: 2.6, h: 0.5, fontSize: 14, fontFace: "Microsoft YaHei", color: "6B7280", align: "center" });
});
```

### 5. Two-Column Comparison

```
┌──────────────────────────────────┐
│ 方法对比 (32pt bold navy)         │
│                                  │
│  ┌─ 方法A ─┐   ┌─ 方法B ─┐     │
│  │ 优势     │   │ 优势     │     │
│  │ 劣势     │   │ 劣势     │     │
│  │ 适用场景 │   │ 适用场景 │     │
│  └─────────┘   └─────────┘     │
│                                  │
└──────────────────────────────────┘
```

### 6. Conclusion Slide

Same structure as title slide but with:
- "总结与展望" or "结论" as the title
- 3-4 key takeaways as bold bullet items
- "感谢聆听 | Q&A" at the bottom

---

## Chart Styling

When adding charts, match the academic navy palette:

```javascript
chartColors: ["1E2761", "CADCFC", "F4A261", "6B7280"],
catAxisLabelColor: "6B7280",
valAxisLabelColor: "6B7280",
valGridLine: { color: "E2E8F0", size: 0.5 },
catGridLine: { style: "none" },
chartArea: { fill: { color: "FFFFFF" } },
```

---

## Text Capacity Rules

These are hard limits to prevent overflow. Chinese text metrics:

- At 16pt in 微软雅黑, each Chinese character ≈ 0.26" wide
- 8.6" usable width fits ~33 Chinese characters per line
- Each line at 16pt with 1.5× line height = 24pt = 0.33"

**Per-slide limits:**

| Element | Max Items | Max Chars Each |
|---------|-----------|----------------|
| Content slide title | 1 | 25 characters |
| Top-level bullet | 5 | 50 characters |
| Sub-bullet | 3 per parent | 60 characters |
| Stat card label | 1 | 10 characters |
| Column header | 1 | 12 characters |

IF content exceeds these limits, add a new slide rather than shrinking fonts or cramming — the presentation is for live delivery, not reading.
