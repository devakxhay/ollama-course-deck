---
marp: true
theme: default
paginate: true
header: 'Building with Ollama: Local LLMs in Action'
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
---

<span class="module-label">Module 08 · Raw Prompt Mechanics</span>

# <!-- fit --> <img src="../assets/ollama.png" height="60" style="vertical-align: middle; margin-right: 15px;" /> FIM & Raw Prompt Mechanics

<div class="title-divider"></div>

### In-Filling Code with Special Token Templates

<!-- 
SPEAKER NOTES (Hinglish):
* Welcome back! Module 07 me humne external APIs dynamically call karne ke liye function calling check kiya.
* Par code autocompleters aur standard code editors me general chat templates inference format corrupt kar sakti hain.
* Aaj hum FIM (Fill-in-the-Middle) algorithms aur raw prompts input interfaces implementation flow check karenge.
* Hum linear code completions aur structured prefix-suffix infilling methods compare karenge.
-->

---

# Append-Only vs. Fill-in-the-Middle: What is the Difference?

<div class="columns">
<div class="col">

### Append-Only Generation
* **Inference Pattern:** Predicts the next token given a prefix context.
* **Limitations:** Only writes code moving forward; ignores code below cursor.
* **Format:** Standard text prompts.

</div>
<div class="col">

### Fill-in-the-Middle (FIM)
* **Inference Pattern:** Fills a gap between a prefix and a suffix context.
* **Capabilities:** Reads surrounding code below the cursor.
* **Format:** Uses special markers: `<fim_prefix>`, `<fim_suffix>`.

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
* Append-Only aur Fill-in-the-Middle parameters differences ko study karte hain.
* Left side standard sequence updates only cursor movement base line par target predict karti hain.
* Right side model suffix text code blocks reads perform karke middle gap output generate karta hai.
-->

---

# How to Configure FIM Requests in Go

```go
prefix := "func add(a, b int) int {\n"
suffix := "\n}"
// 1. Structure FIM prompt using special tokens
fimPrompt := fmt.Sprintf("<fim_prefix>%s<fim_suffix>%s<fim_middle>", prefix, suffix)

payload := map[string]interface{}{
    "model":  "qwen2.5-coder:7b",
    "prompt": fimPrompt,
    "raw":    true, // Bypass default model chat formatting templates
    "stream": false,
}
// ... Send HTTP POST request to /api/generate ...
```

<!-- 
SPEAKER NOTES (Hinglish):
* Go program templates FIM prompt formatting structures details check karein.
* Prefix-suffix components custom tags configurations strings me join kiye jate hain.
* Request body parameter option raw set properties default formats skip update verify karta hai.
-->

---

# How to Read Raw Prompt Telemetry

<div class="columns">
<div class="col">

### Telemetry Payload
```json
{
  "model": "qwen2.5-coder:7b",
  "response": "\treturn a + b",
  "done": true
}
```

</div>
<div class="col">

### Telemetry Check
* <span class="chip">response</span> **Clean Return** — Generates only the code inside the gap, without chat preambles.
* <span class="chip">raw: true</span> **Direct Mode** — Skips wrapping the input in System/User chat tags.

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
* Telemetry response metrics raw parameters verify options evaluate check karein.
* Response body content direct gap code block contents return formats capture karta hai.
* System raw override default chat structures properties direct compute setup verify checks execute karta hai.
-->

---

# Common Problems and How to Fix Them

<div class="callout">
  <strong>📉 Generation Runaways (Infinite Output)</strong>
  The model may keep writing past the suffix block, generating duplicate or garbage code.<br>
  <span class="fix">→ Fix: Add options parameter <code>"stop": ["&lt;fim_suffix&gt;", "\n\n", "}"]</code> in the request payload.</span>
</div>

<div class="callout">
  <strong>🚫 Token Format Mismatch</strong>
  Different code models use different special tokens (e.g., Qwen uses <code>&lt;fim_prefix&gt;</code>).<br>
  <span class="fix">→ Fix: Check the model's metadata to align the correct token tags before sending.</span>
</div>

<div class="callout">
  <strong>⚠️ Template Wrapping Corruption</strong>
  Forgetting raw option wraps special tags inside chat templates, breaking model inference.<br>
  <span class="fix">→ Fix: Always set <code>"raw": true</code> when executing FIM prompt structures.</span>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
* Production deployment setups raw gotchas control limits check karein.
* Runaway output generation problems stop keywords array configurations limits trigger save options handle karein.
* Models tags mismatch specifications check parameters alignment configurations setup verify standard karein.
-->

---

# Next Up: Embeddings & Semantic Search

<div class="title-divider"></div>

### Next Up: [Module 09 · Embeddings](./09-embeddings.html)

* **What we will cover next:**
  * Translating text representations into mathematical vector embeddings
  * Differentiating literal keyword match from conceptual semantic search
  * Extracting high-dimensional float arrays via `/api/embeddings` in Go
  * Overcoming vector size conflicts and chunking loss gotchas
* **Get ready to build semantic search engines and RAG workflows!**

<!-- 
SPEAKER NOTES (Hinglish):
* FIM algorithms aur raw prompt settings override setup successfully completed.
* Lekin local files matching aur semantic operations direct text comparisons logic block par build nahi kiye ja sakte.
* Agle module (Module 09) me hum document semantic similarities search aur text vector representation methods check karenge. Let's move!
-->

