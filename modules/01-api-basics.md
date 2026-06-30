---
marp: true
theme: default
paginate: true
header: 'Ollama in Your Project'
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
    padding: 1rem;
    box-shadow: 0 10px 30px -10px rgba(0, 0, 0, 0.7);
  }
  pre code {
    background: transparent !important;
    color: #f8fafc !important;
    font-size: 0.85em;
    line-height: 1.6;
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

<span class="module-label">Module 01 · REST API Internals</span>

# <!-- fit --> Ollama API Architecture

<div class="title-divider"></div>

### `/api/generate` vs `/api/chat` — Production Mechanics

<!-- 
SPEAKER NOTES (Hinglish):
Hello engineers! Agar aap local LLMs ko apne full-stack core backend controllers me integrate kar rahe ho, toh standard desktop clients ya WebUI ka use karna band karo. 
Ollama background me ek persistent daemon architecture ke upar run hota hai, aur use code se target karne ke liye hamare paas do foundational REST endpoints hain: /api/generate aur /api/chat. 
Aaj hum in dono pipelines ke core differences aur execution mechanics ko line-by-line breakdown karenge.
-->

---

# The Core Architectural Division

<div class="columns">
<div class="col">

### `/api/generate`
* **Execution Pattern:** Single-turn atomic completion.
* **Input Layer:** Flat prompt string.
* **State Management:** Fully stateless transaction.
* **Best Used For:** Automated parsing, strict text classification, code snippet writing.

</div>
<div class="col">

### `/api/chat`
* **Execution Pattern:** Stateful multi-turn loops.
* **Input Layer:** Structured message arrays.
* **State Management:** Context managed via appended history.
* **Best Used For:** Interactive conversational pipelines, stateful agents.

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
Ab screen par dono pipelines ka architectural structural division dekho. 
Left side me hai generate endpoint—ye bilkul simple stateless worker hai. Ek clear task bheja, isne direct completion kiya, kaam khtam. Ko memory store nahi hoti. 
Right side me hai chat endpoint—ye stateful logic design ke liye bana hai, jahan structural conversational hierarchy maintenance zaroori hai. Iske bina multi-turn conversations handle nahi kiye ja sakte.
-->

---

# Payload Blueprint <span class="badge">/api/generate</span>

```json
{
  "model": "qwen2.5-coder:7b",
  "prompt": "Write a Python function to check if a number is prime.",
  "stream": false,
  "options": {
    "temperature": 0.2
  }
}
```

<!-- 
SPEAKER NOTES (Hinglish):
Ye generate endpoint ka simple raw payload blueprint hai. 
Model specifying, stream ko false karna (agar simple atomic completion chahiye), aur options context parameters define karna seekhein.
-->

---

# Payload Blueprint <span class="badge">/api/chat</span>

```json
{
  "model": "qwen2.5-coder:7b",
  "messages": [
    {
      "role": "system",
      "content": "You are a senior code reviewer."
    },
    {
      "role": "user",
      "content": "Is this prime checker function optimal?"
    }
  ],
  "stream": false
}
```

<!-- 
SPEAKER NOTES (Hinglish):
Ab chat API ka schema dekho. Yahan simple prompt string ke bajaye ek structured messages array jata hai. 
Har message me ek role hota hai—jaise system, user, ya assistant—aur unka content. 
Ye message array pure conversational state ko build karne aur model ko behavior prompt dene ke kaam aata hai.
-->

---

# Execution Telemetry: Output Performance Metrics

<div class="columns">
<div class="col">

```json
{
  "model": "qwen2.5-coder:7b",
  "created_at": "2026-06-29T16:30:00Z",
  "response": "...",
  "done": true,
  "total_duration": 1823450600,
  "load_duration": 1502300,
  "prompt_eval_count": 14,
  "prompt_eval_duration": 125000000,
  "eval_count": 48,
  "eval_duration": 1548000000
}
```

</div>
<div class="col">

* <span class="chip">load_duration</span> **VRAM Load Time** — Cold start overhead when model is not active in VRAM.

* <span class="chip">prompt_eval_count / duration</span> **Prompt Speed** — Tokens/sec = `count / duration × 1e9`.

* <span class="chip">eval_count</span> **Generation Throughput** — Actual inference speed of the model.

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
Response payload ko closely check karo, niche telemetry metrics diye hain. 
Total execution time total_duration me nano-seconds me aata hai. 
Cold starts ke time load_duration ka VRAM overhead control karna hota hai. 
Agar load_duration high hai, iska matlab hai model memory me persistent nahi tha. 
Inference throughput ko analyze karne ke liye evaluation count ko eval_duration se calculate karte hain.
-->

---

# Production Gotchas & Error Resolution

<div class="callout">
  <strong>⚡ VRAM Eviction Timeout</strong>
  Ollama unloads models after 5 min of inactivity (<code>OLLAMA_KEEP_ALIVE=5m</code>).<br>
  <span class="fix">→ Fix: Set <code>OLLAMA_KEEP_ALIVE=-1</code> to keep the model loaded indefinitely.</span>
</div>

<div class="callout">
  <strong>📏 Context Length Limits</strong>
  Default context window is 2048 tokens. Long chat histories get truncated silently.<br>
  <span class="fix">→ Fix: Explicitly set <code>"num_ctx": 8192</code> in the <code>options</code> block.</span>
</div>

<div class="callout">
  <strong>🔀 Concurrency Bottleneck</strong>
  Single worker queue defaults to serial execution under load.<br>
  <span class="fix">→ Fix: Set <code>OLLAMA_NUM_PARALLEL=4</code> to enable concurrent stream processing.</span>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
Production deployment me ye gotchas aapka experience kharab kar sakte hain. 
Default behavior me 5 mins inactive rehne par model memory se offload ho jata hai, jisse next request par heavy cold start loading overhead face karna padta hai.
Ise fix karne ke liye environment me OLLAMA_KEEP_ALIVE ko minus one set karo.
Context window support defaults to 2048 tokens, large payloads and history preserve karne ke liye options object me num_ctx parameter manually provide karna zaroori hai.
Concurrency handling ke liye, hardware resource availability ke according OLLAMA_NUM_PARALLEL set karna zaroori hai.
-->