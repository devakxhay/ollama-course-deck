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

<span class="module-label">Module 01 · REST API Internals</span>

# <!-- fit --> <img src="../assets/ollama.png" height="60" style="vertical-align: middle; margin-right: 15px;" /> How Ollama APIs Work

<div class="title-divider"></div>

### `/api/generate` vs `/api/chat` — Under the Hood

<!-- 
SPEAKER NOTES (Hinglish):
* Welcome back! Module 00 me humne overview liya, ab hum actual code integration ki taraf badhenge.
* Local application backend ke liye hum generic visual web clients ya desktop apps par rely nahi karenge.
* Ollama background engine dynamic endpoints expose karta hai: `/api/generate` aur `/api/chat`.
* Aaj hum in dono ke internal payloads aur difference ko detail me analyze karenge.
-->

---

# Chat vs. Generate: What is the Difference?

<div class="columns">
<div class="col">

### `/api/generate`
* **How it works:** One-time request and response.
* **Input:** A simple text prompt.
* **Memory:** No memory. Every request is fresh.
* **Best for:** Simple tasks like reading data or sorting text.

</div>
<div class="col">

### `/api/chat`
* **How it works:** Back-and-forth conversation.
* **Input:** A list of messages with roles (system, user).
* **Memory:** You must send the history every time.
* **Best for:** Building chatbots and AI agents.

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
* Dono endpoints ke structural designs me core architectural difference hai.
* Left side par `/api/generate` pipeline flat requests ko target karti hai aur koi conversational history retain nahi karti.
* Right side par `/api/chat` setup complex messaging workflows ke liye customized role structures array use karta hai.
-->

---

# How the <span class="badge">/api/generate</span> Request Looks 

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
* `/api/generate` payload execution ka straight structure dekhein.
* Yahan simple parameter configuration options ke sath direct raw prompt string input format pass kiya hai.
* Humne low temperature values select ki hain taaki code suggestions predictable aur strict generated logic ke sath mil sakein.
-->

---

# How the <span class="badge">/api/chat</span> Request Looks 

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
* `/api/chat` call ka data payload input dynamic arrays contain karta hai.
* Roles parameter options jaise system instructions aur user entries dynamically specify kiye jate hain.
* Model inputs dynamically update hotey hain multi-turn history keep-alive and mapping features check karne ke liye.
-->

---

# How to Read Performance Numbers

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

* <span class="chip">load_duration</span> **Loading Time** — Time taken to load the model into memory.

* <span class="chip">prompt_eval_count / duration</span> **Reading Speed** — How fast the system reads your prompt.

* <span class="chip">eval_count</span> **Writing Speed** — How fast the model generates new words.

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
* Telemetry response parameters runtime operations check parameters target karte hain.
* Load duration latency batata hai ki model loading processes VRAM cache initialization state load limits me kitna lagta hai.
* Prompt eval parameter prompt evaluation limits trace batata hai jabki eval status baseline speed output rate tokens/sec calculations evaluate karta hai.
-->

---

# Common Problems and How to Fix Them

<div class="callout">
  <strong>⚡ Model Unloads Automatically</strong>
  Ollama removes models from memory after 5 minutes of no use.<br>
  <span class="fix">→ Fix: Set <code>OLLAMA_KEEP_ALIVE=-1</code> to keep the model loaded forever.</span>
</div>

<div class="callout">
  <strong>📏 Long Messages Get Cut Off</strong>
  The default memory size is small. Large chats will lose older messages.<br>
  <span class="fix">→ Fix: Add <code>"num_ctx": 8192</code> inside options.</span>
</div>

<div class="callout">
  <strong>🔀 Requests Wait in Line</strong>
  Ollama handles requests one by one, slowing down multiple users.<br>
  <span class="fix">→ Fix: Set env configuration <code>OLLAMA_NUM_PARALLEL=4</code> to run together.</span>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
* Production setups deploy karte time configurations limits parameters analyze karein.
* Dynamic timeout behavior manage karne ke liye model keep_alive parameters minus one override config options default set karein.
* Baseline context memory limitation parameters default sizes badha kar num_ctx aur dynamic high concurrent request handling settings scale update verify karein.
-->

---

# Next Up: Streaming Responses

<div class="title-divider"></div>

### Next Up: [Module 02 · Streaming Responses](./02-streaming.html)

* **What we will cover next:**
  * Understanding TTFB and latency bottlenecks
  * Setting `"stream": true` and streaming raw JSON chunks
  * Handling line-by-line streaming in Go
  * Handling Nginx proxy buffering & network drops
* **Get ready to fix the latency in your apps!**

<!-- 
SPEAKER NOTES (Hinglish):
* Is module me humne REST base structures aur performance parameter reading trace complete kar liya hai.
* Lekin production systems me long execution queries perceived latency badha sakti hain.
* Agle module (Module 02) me hum time-to-first-token delay trace bypass karne ke liye response streaming set methods check karenge. Let's proceed!
-->