# Course Framework & Rulebook: "Ollama in Your Project"

## 1. Technical Persona & Presentation Strategy
*   **Target Audience:** Software developers, DevOps engineers, and system architects.
*   **Tone:** Production-oriented, high density, authoritative, zero conversational fluff.
*   **Content Philosophy:** Focus strictly on backend logic, raw performance metrics, and configuration code over surface-level UI configurations.

## 2. Language Architecture (The Hinglish Strategy)
*   **Visual Slide Content:** 100% Technical English. Use scannable headers, precise code terminology, and minimalist bullet layouts.
*   **Speaker Notes (Hidden via Comments):** Pure Hinglish following a "Code in English, Explanations in Hinglish" delivery style. Keep descriptions conversational but accurate. Use standard engineering loan words (*payload, buffer, request, overhead*).

## 3. Visual & Layout Guidelines (Marp Standards)
*   Each presentation file must begin with standard global directives: `marp: true`, `paginate: true`, and clean presentation themes.
*   **Slide Dividers:** Separate slide boundaries strictly using a single three-dash sequence (`---`).
*   **Code Containers:** Wrap all syntax implementations inside Markdown blocks with explicit language highlighting tags (e.g., `bash`, `json`). Ensure code formatting stays clean without overflowing visual boundaries.
*   **Structure Sequence:**
    1. Title & Hook Slide
    2. Architecture Core Concept
    3. Payload Configuration / Code Snippet
    4. Execution Telemetry / Output Analysis
    5. Production Gotchas / Error Resolution
*   **Slide Heading Style Consistency:** Maintain consistent slide headings across all modules, drawing from the formats established in Module 00 and Module 01 (e.g., using `[A] vs. [B]: What is the Difference?` for core concepts, `How to [Action]...` for implementations, `How to Read [Telemetry]...` for output analysis, and `Common Problems and How to Fix Them` for production gotchas).


## 4. Marp Frontmatter CSS Standard (MANDATORY)
Every Marp slide file (`.md`) in the `modules/` folder MUST include the following `style:` block in the YAML frontmatter.
This is required to fix the **white background bug on code blocks** — Marp does not reliably resolve CSS custom properties (`var()`) from `@import`-ed files, so all dark-theme values must be hard-coded directly.

```yaml
style: |
  @import "../templates/theme.css";
  section {
    background: #0D1117;
    color: #F5F0E8;
    position: relative;
    padding-left: 3rem;
  }
  section::before {
    content: "";
    position: absolute;
    top: 0; left: 0;
    width: 4px; height: 100%;
    background: #E8593C;
    border-radius: 0 2px 2px 0;
  }
  h1, h2, h3 {
    color: #E8593C;
  }
  footer, header {
    color: #8B949E;
  }
  .columns {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 0;
  }
  .col {
    padding: 0 1.25rem;
  }
  .col:first-child {
    border-right: 1px solid #21262D;
    padding-left: 0;
  }
  pre {
    background: #161B22 !important;
    border: 1px solid #21262D !important;
    border-radius: 8px;
    padding: 0.5rem;
    box-shadow: 0 10px 30px -10px rgba(0, 0, 0, 0.7);
  }
  pre code {
    background: transparent !important;
    color: #f8fafc !important;
    font-size: 0.75em;
    line-height: 1.5;
  }
  code {
    background: #161B22 !important;
    color: #F5F0E8 !important;
    border-radius: 4px;
    padding: 0.1em 0.3em;
  }
  .badge {
    background: rgba(232, 89, 60, 0.12);
    border: 1px solid #E8593C;
    color: #E8593C;
    border-radius: 6px;
    padding: 0.1em 0.55em;
    font-size: 0.65em;
    font-family: 'JetBrains Mono', monospace;
    vertical-align: middle;
    letter-spacing: 0.02em;
  }
  .module-label {
    font-size: 0.7rem;
    font-weight: 700;
    letter-spacing: 0.15em;
    text-transform: uppercase;
    color: #E8593C;
    margin-bottom: 0.5rem;
    display: block;
  }
  .title-divider {
    width: 48px;
    height: 3px;
    background: #E8593C;
    border-radius: 2px;
    margin: 1rem 0;
  }
  .callout {
    border-left: 3px solid #E8593C;
    background: #161B22;
    padding: 0.6rem 1rem;
    border-radius: 0 6px 6px 0;
    margin-bottom: 0.65rem;
    font-size: 0.88em;
    line-height: 1.5;
  }
  .callout strong {
    color: #E8593C;
    display: block;
    margin-bottom: 0.2rem;
  }
  .callout .fix {
    color: #8B949E;
    font-size: 0.9em;
  }
  .callout .fix code {
    font-size: 0.85em;
  }
  .chip {
    display: inline-block;
    background: rgba(232, 89, 60, 0.10);
    border: 1px solid #21262D;
    color: #F5F0E8;
    border-radius: 4px;
    padding: 0.05em 0.45em;
    font-size: 0.78em;
    font-family: 'JetBrains Mono', monospace;
    vertical-align: middle;
  }
```

**Color Reference (matches `templates/theme.css`):**

| Token             | Hex Value |
|-------------------|-----------|
| `--color-bg`      | `#0D1117` |
| `--color-surface` | `#161B22` |
| `--color-text`    | `#F5F0E8` |
| `--color-accent`  | `#E8593C` |
| `--color-muted`   | `#8B949E` |
| `--color-border`  | `#21262D` |

When creating **any new module** or editing the frontmatter of an existing one, always apply this full `style:` block verbatim. Never replace it with `var()` references — they will not resolve and will cause a white background regression.