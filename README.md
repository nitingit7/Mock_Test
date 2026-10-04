# Mock_Test
# Banking Mock Test Platform – JSON Format Guide

This guide documents the complete JSON schema and formatting specifications for creating mock tests compatible with the TCS iON / IBPS clone engine.

---

## 1. Top-Level Structure

A test file consists of a single JSON object with `meta` and `sections`:

```json
{
  "meta": {
    "examName": "SBI Clerk Prelims",
    "testName": "Quant Sample with DI",
    "testType": "sectional",
    "durationMinutes": 20,
    "marksPerQuestion": 1,
    "negativeMarking": 0.25,
    "sectionTimed": false,
    "allowReattempt": true
  },
  "sections": [
    {
      "name": "Quantitative Aptitude",
      "durationMinutes": 20,
      "cutoff": 15,
      "groups": [ ... ],
      "questions": [ ... ]
    }
  ]
}
```

---

## 2. Meta Configuration Fields

| Field | Type | Required | Description |
|---|---|---|---|
| `examName` | String | **Yes** | Name of the exam (e.g. `"SBI Clerk Prelims"`, `"IBPS PO"`). Displayed in header. |
| `testName` | String | **Yes** | Name/title of the specific test (e.g. `"Quant Sample with DI"`). |
| `testType` | String | No | `"sectional"` or `"full"`. If omitted, automatically inferred: `"sectional"` if 1 section, `"full"` if >1 sections. |
| `durationMinutes` | Number | No | Total test duration in minutes. Default: `20` for sectional, `60` for full test. |
| `marksPerQuestion` | Number | No | Default positive marks awarded per question. Default: `1`. |
| `negativeMarking` | Number | No | Default marks deducted for an incorrect response. Default: `0.25`. |
| `sectionTimed` | Boolean | No | For full tests: `true` locks candidates into the current section until time expires or they confirm submitting the section. Default: `true` for full, `false` for sectional. |
| `allowReattempt` | Boolean | No | When `true`, enables Re-attempt mode from results and analysis. Default: `true`. |

---

## 3. Sections & Groups (`groups`)

Each section has a name, duration, and optional question groups:

```json
{
  "name": "Quantitative Aptitude",
  "durationMinutes": 20,
  "cutoff": 15,
  "groups": [
    {
      "id": "t1",
      "passage": "<b>Directions (Q. 6-9):</b>\\nStudy the table below and answer the questions.\\n\\n<table><tr><th>Year</th><th>Revenue (₹ crore)</th><th>Expenses (₹ crore)</th></tr><tr><td>2019</td><td>120</td><td>90</td></tr><tr><td>2020</td><td>150</td><td>100</td></tr></table>"
    },
    {
      "id": "bar1",
      "passage": "<b>Directions (Q. 10-12):</b>\\nThe bar chart shows the production of five companies. Study it and answer the questions.",
      "image": "quant1-di1-bar.png"
    }
  ],
  "questions": [ ... ]
}
```

### Group Attributes
- `id` (String, required): Unique identifier for the group within the section.
- `passage` (String, optional): Context instructions, puzzle rules, passage text, or HTML tables.
- `image` (String, optional): Attached image filename (e.g. `"quant1-di1-bar.png"`). Renders in the left pane after directions text and before any table.

> **Two-Column Split Layout:** Questions that reference a `groupId` automatically display in the authentic two-column split layout: the left pane shows the directions, chart, or table, while the right pane contains the question and its options.

---

## 4. Questions Structure

```json
{
  "id": 10,
  "groupId": "bar1",
  "question": "What is the total production of Companies A and C together in 2023 (in thousand units)?",
  "options": [
    "90",
    "100",
    "110",
    "120",
    "130"
  ],
  "correct": 3,
  "explanation": "50 + 60 = 110 thousand units.<br><b>Full solution:</b> Quant_Sample.pdf, page 10"
}
```

### Question Attributes
- `id` (Number, required): Unique numeric question identifier across the whole test.
- `groupId` (String, optional): References the group `id`. The question inherits the group's passage and image.
- `question` (String, required): Question text. Supports newlines (`\n`), bold tags (`<b>`, `<strong>`, `**text**`), and special characters.
- `options` (Array of Strings, required): Array of at least 2 choices.
- `correct` (Number, required): **1-based index** indicating the correct option (e.g. `1` for the first option, `3` for the third).
- `marks` (Number, optional): Positive marks override for this question (defaults to `meta.marksPerQuestion`).
- `negative` (Number, optional): Penalty marks override for this question (defaults to `meta.negativeMarking`).
- `image` (String, optional): Question-specific diagram filename (for standalone questions or question-specific figures).
- `explanation` (String, optional): Short solution text shown in Analyse mode and Re-attempt mode (after answering). May include PDF book reference in the text (e.g. `"<br><b>Full solution:</b> Quant_Sample.pdf, page 10"`).

---

## 5. Adding Charts and Tables

### HTML Tables in Passages
Embed HTML `<table>` tags directly inside group `passage` strings. The renderer styles them with crisp borders, alternating headers, and horizontal scrollbars that prevent page distortion:

```html
<table>
  <tr><th>Year</th><th>Revenue (₹ crore)</th><th>Expenses (₹ crore)</th></tr>
  <tr><td>2021</td><td>180</td><td>125</td></tr>
  <tr><td>2022</td><td>210</td><td>150</td></tr>
</table>
```

### Chart Images
1. Reference the exact image file name in `group.image` or `question.image`:
   ```json
   "image": "quant1-di1-bar.png"
   ```
2. On the landing page, click the **"🖼️ Add images"** button to select your image files alongside the JSON. You can also drag-and-drop JSON and image files directly onto the page.
3. The platform displays a live image checklist showing `✔ Found` or `⚠️ Missing` for every image mentioned in your test.
4. Images scale responsively, keep their aspect ratio, and feature **click-to-enlarge** (lightbox modal) without exiting exam fullscreen.
5. Missing images show a friendly placeholder box: `"Image not found: <filename>. Attach it with the Add images button."`

> **Recommended Image Guidelines:**
> - Format: PNG or WebP
> - Dimensions: ~800px wide
> - File size: Under 150 KB each for optimal storage efficiency
> - File naming: Use lowercase filenames without spaces (e.g., `di1_chart.png`).

---

## 6. Text Formatting Rules

- **Line breaks:** `\n` or `\\n` creates a line break (`<br>`).
- **Paragraphs:** Double newline `\n\n` creates a clean `0.6em` paragraph gap.
- **Bold text:** Use standard HTML `<b>text</b>`, `<strong>text</strong>`, or Markdown `**text**`.
- **Formatting tags allowed:** `<b>`, `<strong>`, `<i>`, `<u>`, `<sub>`, `<sup>`, `<br>`, `<p>`, `<span>`, `<div>`, `<table>`, `<tr>`, `<th>`, `<td>`.
- **Symbols:** Mathematical symbols, fractions, rupee symbol `₹`, arrows, and Greek symbols are fully supported.

---

## 7. Common Mistakes to Avoid

1. **0-based `correct` index:** `correct` must be 1-based (`1`, `2`, `3`, `4`, or `5`). Using `0` will fail validation.
2. **Trailing commas:** JSON strictly prohibits trailing commas after the last item in arrays or objects.
3. **Curly quotes:** Ensure all keys and strings use straight quotes (`"key"`) and not curly quotes (`“key”`).
4. **Raw unescaped line breaks:** In JSON strings, line breaks must be written as `\n`, never raw newlines across lines.
5. **Image filename mismatch:** Filenames are case-sensitive. `"Chart1.png"` will not match `"chart1.png"`.
6. **Forgetting to attach images:** Remember to attach your images using **"Add images"** or select JSON and images together.

