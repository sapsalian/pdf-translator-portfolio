# PDF Translator

> 🇰🇷 [한국어 버전 보기](./README.ko.md)

An AI-powered PDF translation tool that parses coordinate-scattered text fragments from fixed-layout PDFs, reconstructs them into human-readable flow, and produces a translated PDF that preserves the original document's full visual and structural fidelity.

---

## Demo

<video src="docs/videos/demo.mov" controls width="100%"></video>

### Before / After

*YOLOv9 paper (arXiv:2402.13616) — first page*

| Original (English) | Translated (Korean) |
|---|---|
| ![Original](docs/images/original-page1.png) | ![Translated](docs/images/translated-page1.png) |

---

## Background

PDF is a **fixed-layout format** — text is stored as coordinate-positioned fragments, not as structured paragraphs. A single visible sentence can be split into dozens of disconnected spans across the file, each carrying only positional metadata.

This makes PDF translation fundamentally different from translating plain text. Simply extracting and feeding raw PDF text to a translation API produces broken, out-of-order results. The core challenge is **reconstructing human reading order from coordinate-based chaos** before any translation begins.

---

## Pipeline Overview

```
PDF Input
    │
    ▼
[Stage 1] Document Analysis
    - Extract coordinate-based text spans per page via PyMuPDF
    - Reconstruct paragraphs by inferring reading flow from positions
    - Pre-extract glossary via GPT for domain term consistency
    │
    ▼
[Stage 2] Preprocessing
    - Classify each block's type with YOLO (Figure / Table / List-item / Formula / etc.)
      to route it to the appropriate processing algorithm
    - Merge fragmented spans into coherent text blocks
    - Sort blocks by inferred reading order (columns, footnotes, captions)
    - Detect and tag inline elements: superscripts, subscripts, style transitions
    - Clean artifacts from PDF parsing (hyphenation, encoding noise)
    │
    ▼
[Stage 3] Translation
    - Translate each block with GPT, supplying glossary + page context
    - Preserve inline style markers (bold, italic, font size shifts)
    - Maintain superscript / subscript positions through translation
    │
    ▼
[Stage 4] PDF Reconstruction
    - Erase original text from each block region
    - Re-render translated text at the original coordinates
    - Restore font family, size, color, weight, and italic style
    - Reattach hyperlinks and functional elements to translated text positions
    - Output as {filename}-ko.pdf
```

---

## What Gets Preserved

| Element | Preserved |
|---|---|
| Paragraph structure and reading order | ✅ |
| Font size, color, weight (bold), italic | ✅ |
| Superscripts and subscripts | ✅ |
| Hyperlinks and functional elements | ✅ |
| Multi-column layout | ✅ |
| Domain term consistency (via glossary) | ✅ |

---

## Technical Challenges

### 1. Reconstructing reading order from coordinate fragments
Fixed-layout PDFs store text as raw coordinate spans — there is no concept of "paragraph" or "sentence" in the file format. The pipeline infers reading flow by analyzing spatial relationships: proximity, alignment, column boundaries, and font size changes. This reconstruction must handle multi-column layouts, footnotes, and figure captions without breaking reading order.

### 2. Preserving inline styles across translation
Text blocks often contain mixed styles within a single sentence (e.g., a bold term mid-paragraph, a superscript citation). Per-character style metadata is extracted before translation and reapplied after, compensating for the length differences that translation introduces.

### 3. Consistent terminology across pages
Technical documents use domain-specific terms with multiple valid translations depending on context. A pre-translation glossary extraction step (via GPT) establishes canonical translations for key terms before the page-by-page pass, ensuring consistency across the entire document.

### 4. Korean font rendering on English-only PDFs
Most English PDFs embed only Latin fonts. The tool identifies the closest system font family to the original and substitutes a Korean-capable equivalent, preserving visual tone while enabling CJK rendering.

### 5. Recalculating line spacing after translation
English and Korean differ significantly in character density — a translated block often produces a different number of lines than the original. Rendering translated text at the original coordinates causes lines to overflow or leave gaps. The pipeline dynamically recalculates line spacing based on the translated line count and the original block's bounding box, redistributing vertical space so the result fits within the reserved region.

---

## Tech Stack

| Category | Technology |
|---|---|
| PDF Processing | PyMuPDF (fitz) |
| Layout Detection | YOLO (Ultralytics) |
| Translation | OpenAI GPT API |
| Concurrency | Python ThreadPoolExecutor |
| Language | Python 3.10+ |

---

## Project Status

This repository contains documentation and demos only. The source code is proprietary (developed during prior employment) and is not publicly available.

---

## License

MIT
