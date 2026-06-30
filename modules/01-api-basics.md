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
    color: var(--color-text) !important;
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

# <!-- fit --> How Ollama APIs Work

<div class="title-divider"></div>

### `/api/generate` vs `/api/chat` — Under the Hood

<!-- 
SPEAKER NOTES (Hinglish):
Hey guys! Agar aap local LLMs ko apne real-world backend architectures me integrate kar rahe ho, toh generic desktop clients ya visual WebUIs se thoda aage badhna hoga.
Ollama background me ek background daemon service ki tarah chalta hai, aur use target karne ke liye code me hamare paas do main pipelines hain: /api/generate aur /api/chat.
Chalo dekhte hain in dono me raw differences kya hain, and code ke end par inhe kaise handle karte hain.
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
Dono endpoints ka main diff unke architectural design me hai.
Left side par dekho: generate pipeline—ye ek task worker ki tarah kaam karta hai. Flat text inputs handle karta hai aur server par context save nahi karta.
Right side par chat pipeline—ye complex conversational workflows ke liye hai jahan system, user aur assistant messages ka chronological history structure maintained rakhna padta hai.
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
Ye `/api/generate` ka flat execution call hai.
Notice karo ki yahan seedha raw prompt pass ho raha hai. Streaming ko humne standard API response style me block kiya hai (`stream: false`) aur low temperature set kiya hai taaki hume solid, deterministic code generation mile.
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
Wahi agar `/api/chat` ka format dekhein, toh structure thoda change ho jata hai.
Yahan raw string input ke bajaye hamara content `messages` array ke form me jata hai.
Isme roles (jaise system behavior, user prompt) specify kiye jate hain, jisse model dynamically response context context-aware templates me convert kar sake.
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
Ollama response me solid performance metrics return karta hai.
Sabse pehle check karo `load_duration`—agar model VRAM me unloaded tha toh heavy loading lagti hai.
`prompt_eval_count` aur duration batate hain ki hamara prompt system ne kis latency rate se parse kiya, aur `eval_count` hume baseline inference throughput details deta hai.
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
Production systems me deploy karte time teen important settings hamesha dhyan me rakhein.
Pehla, 5-minute timeout pe model memory se unload ho jata hai jiski wajah se first-hit call me bottleneck aata hai. Iske liye keep_alive environmental parameter minus one override set karein.
Dusra, memory threshold by default standard 2k par capped hai, isliye options me num_ctx parameter bada set karein.
Teesra, high concurrent traffic serve karne ke liye OLLAMA_NUM_PARALLEL configuration ka parallel execution set karna mat bhoolna.
-->