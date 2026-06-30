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

<span class="module-label">Module 06 · Multimodal Vision</span>

# <!-- fit --> <img src="../assets/ollama.png" height="60" style="vertical-align: middle; margin-right: 15px;" /> Multimodal Vision Input

<div class="title-divider"></div>

### Processing Visual Data with Local Models

<!-- 
SPEAKER NOTES (Hinglish):
* Welcome back! Module 05 me humne conversational state aur memory context progress check kiya.
* Par text queues ke alawa modern applications me images aur visual assets process karna mandatory step hai.
* Aaj hum seekhenge ki base64 formatting options specify karke multi-modal datasets kaise target karte hain.
* Hum textual tokenizers aur vision model components ke primary processing differences check karenge.
-->

---

# Text Models vs. Vision Models: What is the Difference?

<div class="columns">
<div class="col">

### Text-Only Models
* **Pipeline:** Tokenizer converts text strings into token IDs and embeddings.
* **Context Space:** Simple text sequences only.
* **Use Case:** Code generation, text summaries, data extraction.

</div>
<div class="col">

### Multimodal Vision Models
* **Pipeline:** Vision Encoder projects image patches/tensors into token embedding space.
* **Context Space:** Combines text tokens and image token arrays.
* **Use Case:** OCR, chart analysis, visual description.

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
* Text-Only aur Multimodal models ke architectural differences ko closely evaluate karein.
* Left side text pipeline raw token sequences space dimensions limits par operate karti hai.
* Right side vision pipeline structural representations features vector map context space me project karti hai.
-->

---

# How to Send Images in Go

```go
// 1. Read local file and encode it to Base64
imgBytes, _ := os.ReadFile("invoice.png")
base64Img := base64.StdEncoding.EncodeToString(imgBytes)

// 2. Prepare payload with base64 string inside images array
payload := map[string]interface{}{
    "model":  "llama3.2-vision",
    "prompt": "Extract the total amount from this invoice.",
    "images": []string{base64Img}, // Pass base64 image data
    "stream": false,
}

// ... Send HTTP POST to /api/generate ...
```

<!-- 
SPEAKER NOTES (Hinglish):
* Go program images integration flows aur base64 parsing details check karein.
* os.ReadFile method target binary files read karke standard encoding array parameters transform karta hai.
* Payload images array base64 bytes dynamically capture settings verify criteria load karta hai.
-->

---

# How to Read Multimodal Telemetry

<div class="columns">
<div class="col">

### Telemetry Payload
```json
{
  "model": "llama3.2-vision",
  "done": true,
  "total_duration": 1540320000,
  "prompt_eval_count": 424,
  "prompt_eval_duration": 820300000
}
```

</div>
<div class="col">

### Telemetry Check
* <span class="chip">prompt_eval_count</span> **Image Grid Tokens** — Images are represented as a large grid of tokens (e.g., 400+ tokens).
* <span class="chip">prompt_eval_duration</span> **Heavy Latency** — Image evaluation is computationally expensive on the GPU.

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
* Multimodal performance parameters aur token telemetry limits analyze karein.
* Input metrics eval count image dimensions complexity ke basis par linearly evaluate hotey hain.
* Vision encoder compute calculations VRAM context memory utilization variables dynamic update level set karti hain.
-->

---

# Common Problems and How to Fix Them

<div class="callout">
  <strong>📏 High Resolution VRAM Blowouts</strong>
  Large, raw images inflate base64 sizes, consuming massive VRAM and freezing GPUs.<br>
  <span class="fix">→ Fix: Compress to JPG and resize images client-side before encoding.</span>
</div>

<div class="callout">
  <strong>🚫 Model Compatibility Errors</strong>
  Sending image payloads to text-only models throws syntax exceptions or gets silent ignores.<br>
  <span class="fix">→ Fix: Verify that your active model supports vision (e.g., <code>llama3.2-vision</code>).</span>
</div>

<div class="callout">
  <strong>⏳ Network Payload Inflation</strong>
  Huge base64 arrays saturate server networks and cause HTTP timeouts.<br>
  <span class="fix">→ Fix: Use smaller image resolutions and implement strict file size limit validation.</span>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
* Vision pipelines gotchas optimization rule settings checks run karein.
* Image compression logic apply karke network weight sizes aur GPU bottlenecks control setups configure check karein.
* Text models image calls limitations safety checks verification options backend structures check criteria run karein.
-->

---

# Next Up: Function Calling & Tool Use

<div class="title-divider"></div>

### Next Up: [Module 07 · Tool Use](./07-tool-use.html)

* **What we will cover next:**
  * Connecting local models to real-time external APIs
  * Differentiating conversational chat responses from tool executions
  * Defining function schemas and properties structures in Go
  * Handling argument hallucinations and tool execution loops
* **Get ready to build active, action-capable AI pipelines!**

<!-- 
SPEAKER NOTES (Hinglish):
* Multimodal vision inputs aur base64 payload configurations setup completed.
* Lekin static inputs analysis passive feedback pipelines tak restricted hai, real action capability check nahi kar sakti.
* Agle module (Module 07) me hum autonomous database updates aur active API executions ke liye Tool Use structures check karenge. Let's proceed!
-->

