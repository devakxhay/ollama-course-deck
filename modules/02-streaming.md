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

<span class="module-label">Module 02 · Streaming Responses</span>

# <!-- fit --> <img src="../assets/ollama.png" height="60" style="vertical-align: middle; margin-right: 15px;" /> Streaming Responses

<div class="title-divider"></div>

### Solving LLM Latency with Streamed Chunks

<!-- 
SPEAKER NOTES (Hinglish):
* Welcome back! Module 01 me humne REST APIs call metrics check kiye the par direct request block me long generation times bottleneck create karte hain.
* Aaj hum perceived latency aur time-to-first-token parameters improve karne ke liye streaming implementation compare karenge.
* Hum response block buffering aur real-time streaming pipelines ke architectural patterns analyze karenge.
-->

---

# Buffering vs. Streaming: What is the Difference?

<div class="columns">
<div class="col">

### Buffered Response (`stream: false`)
* **Inference Pipeline:** Wait for the model to finish generating the entire response.
* **Network Payload:** One large JSON packet sent after completion.
* **Perceived Latency:** High. User sees a blank loading indicator for seconds.

</div>
<div class="col">

### Streamed Response (`stream: true`)
* **Inference Pipeline:** Yield tokens instantly as they are computed by the GPU.
* **Network Pipeline:** HTTP Chunked Transfer Encoding streams JSON lines.
* **Perceived Latency:** Low. Characters appear on-screen in real-time.

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
* Standard buffering aur dynamic response streaming ka structural setup analyze karein.
* Standard flow me client completely wait karega jab tak model end-of-sequence token encounter nahi kar leta.
* Streaming setup activation se hardware memory generation chunks line-by-line real-time me push hone lagte hain.
-->

---

# How to Stream Responses in Go

```go
resp, err := http.Post("http://localhost:11434/api/generate", "application/json", bytes.NewBuffer(payload))
if err != nil {
    log.Fatalf("Request failed: %v", err)
}
defer resp.Body.Close()

scanner := bufio.NewScanner(resp.Body)
for scanner.Scan() {
    var chunk GenerateResponse
    if err := json.Unmarshal(scanner.Bytes(), &chunk); err != nil {
        log.Printf("Failed to decode chunk: %v", err)
        continue
    }
    fmt.Print(chunk.Response) // Print token in real-time
}
```

<!-- 
SPEAKER NOTES (Hinglish):
* Backend connection setup aur response body data stream process flow check karein.
* Go buffer memory scanner use karke hum chunks dynamically read karte hain.
* Newline characters scan parsing loop apply karke output console par feed kiya jata hai.
-->

---

# How to Read Streamed Telemetry

```json
{"model":"qwen2.5-coder:7b","created_at":"...","response":"Hello","done":false}
{"model":"qwen2.5-coder:7b","created_at":"...","response":" world","done":false}
{"model":"qwen2.5-coder:7b","created_at":"...","response":"!","done":false}
{
  "model":"qwen2.5-coder:7b",
  "done":true,
  "total_duration": 450230000,
  "load_duration": 850200,
  "prompt_eval_count": 12,
  "eval_count": 3
}
```

* <span class="chip">done: false</span> **Intermediate Chunks** — Contain partial generation text.
* <span class="chip">done: true</span> **Terminal Chunk** — Empty content, returns final telemetry metrics.

<!-- 
SPEAKER NOTES (Hinglish):
* Raw telemetry endpoints return formats dynamic responses specify karte hain.
* Har early token generation payload object `done: false` state and text string returns execute karta hai.
* Final message packet boundary reach hone par `done: true` dynamic context metrics metadata payload block load karta hai.
-->

---

# Common Problems and How to Fix Them

<div class="callout">
  <strong>🔄 Reverse Proxy Buffering</strong>
  NGINX or Cloudflare might buffer the stream, rendering client-side updates synchronous.<br>
  <span class="fix">→ Fix: Add header <code>X-Accel-Buffering: no</code> and configure flush intervals.</span>
</div>

<div class="callout">
  <strong>⚠️ Connection Drops and Retries</strong>
  Network glitches mid-stream corrupt state. The client loses the context of generated tokens.<br>
  <span class="fix">→ Fix: Implement a retry-offset token buffer or fail-safe transaction log on the server.</span>
</div>

<div class="callout">
  <strong>🛠 Incomplete JSON Lines</strong>
  Scanning before a line completes can result in half-read JSON strings.<br>
  <span class="fix">→ Fix: Use standard delimiter scanning (newline <code>\n</code>) rather than reading fixed byte sizes.</span>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
* Production load deployments limits problems control methods verify karein.
* Proxies settings update rule setting headers control jaise `X-Accel-Buffering` parameter default configure set karein.
* Memory packet drops resolve logic offsets aur standard scan buffer boundaries carefully check karein.
-->

---

# Next Up: Structured Output (JSON Mode)

<div class="title-divider"></div>

### Next Up: [Module 03 · Structured Output (JSON Mode)](./03-json-mode.html)

* **What we will cover next:**
  * Setting `"format": "json"` in the API request body
  * Forcing schema-conforming token generations
  * Unmarshaling dynamic string JSON responses in Go
  * Handling validation, prompt key requirements, and schema hallucinations
* **Get ready to build reliable program pipelines!**

<!-- 
SPEAKER NOTES (Hinglish):
* Humne raw response streaming setups successfully complete target evaluate kiya hai.
* Par custom backend APIs build karte time dynamic plain strings standard pipelines me error create kar sakti hain.
* Agle module (Module 03) me hum output format schema conform JSON mode structures trigger methods check karenge. Let's move on!
-->

