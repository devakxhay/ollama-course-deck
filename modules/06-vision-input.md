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

# <!-- fit --> Multimodal Vision Input

<div class="title-divider"></div>

### Processing Visual Data with Local Models

<!-- 
SPEAKER NOTES (Hinglish):
Hey guys! Welcome back. Aaj hum baat karenge local systems me ek bohot hi powerful upgrade ke baare me—wo hai Multimodal Vision Input.
Ab tak hum sirf textual queries handle kar rahe the, lekin modern local models jaise llama3.2-vision ya qwen2-vl directly raw images ko input ki tarah accept kar sakte hain.
Is module me hum seekhenge ki backend code me images ko base64 format me encode karke model tak kaise push kiya jata hai.
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
In dono architectures ke dynamic structure differences ko check karte hain.
Left side par dekhiye normal text-only models—yahan user input standard tokenizer ke through tokens me break hokar compute hota hai.
Right side par check karein vision models—yahan model ke paas ek extra component hota hai jise Vision Encoder bolte hain. Ye image layers ko small patches me divide karke, unhe direct LLM ke dimensional token space me map kar deta hai.
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
Go codebase implementation par focus karte hain.
Pehle hum raw image file ko `os.ReadFile` se byte array me load karte hain, aur use base64 standard string format me convert kar dete hain.
Request payload me notice karein: humne model parameters me llama3.2-vision specify kiya hai, aur images map structure array property me base64 variable pass kiya hai. Ye array dynamic multiple image uploads support karta hai.
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
Vision metrics and telemetry values ko evaluate karte hain.
Left side response check karein: prompt_eval_count kafi high dikhega (424 tokens), jabki user prompt sirf single line text instructions tha.
Right side par logic verify karein: aesa isliye hai kyunki Vision Encoder image structure size ke basis par use abstract token representations (patches) me map kar deta hai. Ye processing visual compute overhead add karti hai, jisse prompt_eval_duration text prompts ke मुकाबले significant scale par badh jata hai.
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
Vision pipelines compile karte time production gotchas ka dhyan rakhein.
Sabse pehle, agar image high-resolution raw file hai toh base64 size badhega aur system memory crash ho jayegi. Isliye client-side par scale down and JPG optimization filters execute karein.
Doosra problem compatibility hai—hamesha target model parameter check karein. Agar model text-only hai toh payload ignore ho jayega, isliye llama3.2-vision model hi target karein.
Teesra network payload overhead hai—ensure dynamic network connections stable ho aur timeouts configurations properly handles ho backend side par.
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
Chalo ab next module par chalte hain! Agle class me hum seekhenge ki kaise functions calling schemas compile karte hain aur APIs aur local database integration triggers implement karte hain. See you in the next module!
-->

