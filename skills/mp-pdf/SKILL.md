---
name: mp-pdf
description: "Use this skill to convert a Markdown file into a professionally styled PDF with mühlemann+popp corporate branding. Handles Mermaid diagram rendering, pandoc HTML conversion, CSS injection (Manrope font, mustard accent), m+p logo header, and Playwright Chromium PDF generation. Trigger when the user asks to generate a PDF, create a branded report, or convert markdown to PDF."
argument-hint: "[markdown-file]"
---

# m+p Branded PDF Generation

Convert a Markdown file into a professionally styled PDF with mühlemann+popp corporate branding.

## Parameters

- **Input file** (required): Path to the Markdown file to convert
- **Output PDF** (optional): Path for the output PDF. Default: same name with `.pdf` extension
- **Title** (optional): Title for the HTML metadata. Default: derived from the first `# heading` in the Markdown

## Instructions

**Strategy:** Convert Markdown → HTML (via pandoc with custom CSS) → PDF (via Playwright Chromium).
This produces high-quality PDFs with proper table styling, colored emojis, and rendered diagrams.
Do NOT use `md-to-pdf` or `wkhtmltopdf` — they rely on Puppeteer/system Chrome which often fails in sandboxed or headless environments, and they render emojis as black-and-white glyphs.

### Step 1: Install required tools (if not already installed)

```bash
# Mermaid CLI for rendering diagrams
npm list -g @mermaid-js/mermaid-cli || npm install -g @mermaid-js/mermaid-cli

# Playwright for PDF generation (uses its own bundled Chromium — no system Chrome needed)
pip3 install --break-system-packages playwright 2>/dev/null
# Install system deps for Playwright's Chromium (needs sudo)
sudo python3 -m playwright install-deps chromium 2>/dev/null
# Download Playwright's bundled Chromium
python3 -m playwright install chromium

# Pandoc for Markdown → HTML conversion
which pandoc || sudo apt-get install -y pandoc
```

### Step 2: Configure Mermaid CLI to use Playwright's Chromium (avoids snap/sandbox issues)

```bash
# Find Playwright's Chromium binary
CHROME_PATH=$(find ~/.cache/ms-playwright -name 'chrome' -path '*/chrome-linux/*' 2>/dev/null | head -1)

# Create puppeteer config for mmdc
cat > /tmp/puppeteer-config.json << EOF
{
  "executablePath": "$CHROME_PATH",
  "args": ["--no-sandbox", "--disable-setuid-sandbox", "--allow-file-access-from-files"]
}
EOF
```

### Step 3: Extract and render Mermaid diagrams

- Use a Python script to find all mermaid code blocks in the input Markdown and save each to `/tmp/diagram-N.mmd`
- For each mermaid block, render to PNG using mmdc with the Playwright Chromium:
  ```bash
  # DISPLAY must be set if running in a VNC/X11 environment
  mmdc -i /tmp/diagram-N.mmd -o {name}-diagram-N.png -b white -p /tmp/puppeteer-config.json
  ```

### Step 4: Replace mermaid blocks with rendered images

- Replace each ` ```mermaid ... ``` ` block with `![Diagram N]({name}-diagram-N.png)`
- Save the modified report to `{name}-with-images.md`

### Step 5: Convert to styled HTML

```bash
pandoc {name}-with-images.md -o {name}.html --standalone \
  --metadata title="{title}"
```

### Step 6: Inject professional CSS into the generated HTML `<style>` block

Design guidelines:
- Font: **Manrope** (Google Fonts) as primary, with Segoe UI / Helvetica Neue fallbacks
- Accent color: **mustard** `hsl(41, 75%, 61%)` (#D4A843) and its shades — used for heading borders, table header backgrounds, blockquote borders, links
- Use **colored emojis** where appropriate (e.g. ⚠️, ✅, 🔴, 🟡, 🟢) — Playwright renders them natively
- Add the **mühlemann+popp corporate logo** in the page header: `https://muehlemann.com/img/muehlemann+popp.svg`

```css
@import url('https://fonts.googleapis.com/css2?family=Manrope:wght@300;400;500;600;700&display=swap');
@page { size: A4; margin: 10mm; }
body {
  font-family: 'Manrope', 'Segoe UI', 'Helvetica Neue', Arial, sans-serif;
  font-size: 11px; line-height: 1.5; color: #1a1a1a;
  max-width: 100%; margin: 0 auto;
}
h1 { font-size: 22px; color: #1a365d; border-bottom: 2px solid hsl(41, 75%, 61%); padding-bottom: 6px; margin-top: 24px; }
h2 { font-size: 17px; color: #2c5282; border-bottom: 1px solid hsl(41, 75%, 75%); padding-bottom: 4px; margin-top: 20px; }
h3 { font-size: 14px; color: #2d3748; margin-top: 16px; }
h4 { font-size: 12px; color: #4a5568; }
table { border-collapse: collapse; width: 100%; margin: 10px 0; font-size: 10.5px; }
th { background-color: hsl(41, 75%, 93%); color: #2d3748; font-weight: 600; text-align: left; padding: 6px 8px; border: 1px solid hsl(41, 75%, 80%); }
td { padding: 5px 8px; border: 1px solid #e2e8f0; vertical-align: top; }
tr:nth-child(even) { background-color: #f7fafc; }
code { background-color: hsl(41, 75%, 95%); padding: 1px 4px; border-radius: 3px; font-size: 10px; }
pre { background-color: #f7fafc; border: 1px solid #e2e8f0; border-radius: 4px; padding: 8px; overflow-x: auto; font-size: 10px; }
pre code { background: none; padding: 0; }
img { max-width: 100%; height: auto; }
hr { border: none; border-top: 1px solid hsl(41, 75%, 80%); margin: 20px 0; }
blockquote { border-left: 3px solid hsl(41, 75%, 61%); margin: 10px 0; padding: 6px 12px; background: hsl(41, 75%, 96%); }
a { color: hsl(41, 75%, 45%); }
header#title-block-header { display: none; }
```

Also inject a **logo header** right after `<body>`:
```html
<div style="text-align: right; margin-bottom: 12px;">
  <img src="https://muehlemann.com/img/muehlemann+popp.svg" alt="mühlemann+popp" style="height: 32px;" />
</div>
```

### Step 7: Generate PDF using Playwright (supports color emojis, proper CSS rendering)

```python
from playwright.sync_api import sync_playwright
import os

html_path = os.path.abspath('{name}.html')

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    page.goto(f'file://{html_path}')
    page.pdf(
        path='{name}.pdf',
        format='A4',
        margin={'top': '10mm', 'bottom': '10mm', 'left': '10mm', 'right': '10mm'},
        print_background=True,
    )
    browser.close()
```

### Step 8: Clean up temporary files

- Remove `/tmp/diagram-*.mmd` and `/tmp/puppeteer-config.json`
- Keep the PNG diagram files alongside the PDF for reference

## Output

1. `{name}.html` — Styled HTML with rendered diagram images
2. `{name}.pdf` — Final PDF with m+p corporate branding
3. `{name}-diagram-N.png` — Rendered Mermaid diagrams (kept for reference)
