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

# <!-- fit --> Streaming Responses

<div class="title-divider"></div>

### Solving LLM Latency with Streamed Chunks

<!-- 
SPEAKER NOTES (Hinglish):
Hey guys! Welcome back. Aaj hum baat karenge ek bohot hi critical production issue ke baare me—aur wo hai latency.
Jab hum kisi LLM se bada response generate karate hain, toh user ko wait karna padta hai jab tak full JSON respond na ho jaye.
Is class me hum seekhenge ki kaise streaming responses ka use karke hum application ki perceived latency ko drop kar sakte hain aur Time to First Token ko optimize kar sakte hain.
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
Chalo iske architecture ko samajhte hain.
Left side par dekho standard buffering: jab `stream: false` hota hai, tab Ollama tab tak wait karta hai jab tak complete generation khatam na ho jaye, and fir single large JSON return karta hai. Isse user ko lagta hai ki app stuck ho gayi hai.
Right side par streaming hai: jab `stream: true` hota hai, toh har ek single token generate hote hi system network chunk ke through deliver karne lagta hai. Client screen par typing effect show kar sakta hai bina waiting state ke.
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
    var chunk struct {
        Response string `json:"response"`
        Done     bool   `json:"done"`
    }
    if err := json.Unmarshal(scanner.Bytes(), &chunk); err != nil {
        log.Printf("Failed to decode chunk: %v", err)
        continue
    }
    fmt.Print(chunk.Response) // Print token in real-time
}
```

<!-- 
SPEAKER NOTES (Hinglish):
Backend logic me stream ko handle karna bohot simple hai.
Yahan hum Go standard library use kar rahe hain. http.Post call ke baad hum poori body ko memory me read karne ke bajaye bufio.NewScanner create karte hain.
scanner.Scan() loop tab tak chalega jab tak network se lines (JSON lines) aati rahengi. Har single line ko parse karke hum print karte rehte hain, jisse real-time output stream generate ho sake.
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
Ab check karte hain raw response payload format.
Inference start hote hi Ollama server dynamic JSON chunks return karta hai. Har intermediate chunk me done: false hota hai aur dynamic token input stream text pass hota hai.
Terminal chunk, yani last JSON object me done: true aata hai aur response text empty hota hai. Yahan execution metadata details jaise load_duration, prompt_eval_count aur generation speed key-values return hote hain.
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
Production deployment ke time streaming me do-teen bade issue aate hain.
Pehle, reverse proxies jaise Nginx default settings par pure response ko buffer karne ki koshish karte hain, jisse streaming work nahi karti. Iske liye X-Accel-Buffering: no response header bhejna zaroori hai.
Doosra, user connections bich me drop ho sakte hain, isliye state-saving fallback layers logic backend par execute hona chahiye.
Aur third, scanning logic hamesha explicit newline boundary scan par based hona chahiye taaki corrupted lines throw na ho.
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
Chalo ab next module par chalte hain! Agle class me hum seekhenge ki kaise custom JSON mode configure kiya jata hai aur outputs ko structured database pipelines ke liye ready kiya jata hai. See you in the next module!
-->

