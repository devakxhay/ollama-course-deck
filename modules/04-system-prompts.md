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

<span class="module-label">Module 04 · System Prompts</span>

# <!-- fit --> <img src="../assets/ollama.png" height="60" style="vertical-align: middle; margin-right: 15px;" /> System Prompts

<div class="title-divider"></div>

### Formatting Assistant Behavior at the System Level

<!-- 
SPEAKER NOTES (Hinglish):
* Welcome back! Module 03 me humne structured outputs validate karne ke liye JSON Mode seekha.
* Par output structure ke sath model ke instructions, guardrails aur behavior control karna bhi zaroori hai.
* Aaj hum seekhenge ki kaise role: system configure karke model ki personality ko standard set kiya jata hai.
* Hum System aur User roles ke semantic differences aur dynamic setups check karenge.
-->

---

# System Role vs. User Role: What is the Difference?

<div class="columns">
<div class="col">

### System Role (`role: system`)
* **Purpose:** Sets the global behavior, guardrails, personality, and tone.
* **Scope:** Persists across the entire chat session.
* **Privilege:** Higher weight; acts as instructions the model must follow.

</div>
<div class="col">

### User Role (`role: user`)
* **Purpose:** Provides the specific task, query, or dynamic input.
* **Scope:** Transient and unique to each individual turn.
* **Privilege:** Evaluated under the constraints defined by the system role.

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
* System Role aur User Role ke configuration differences ko samajhte hain.
* System configuration ek constitution ki tarah instructions register karti hai jo output generation constraints specify karti hai.
* User messages real-time query inputs execute karte hain, jo system rules standard priority scale par execute hotey hain.
-->

---

# How to Configure System Prompts in Go

```go
// 1. Build the messages array with system and user roles
messages := []map[string]string{
    {"role":    "system", "content": "You are a database admin. Answer ONLY with raw SQL queries.",},
    {"role":    "user", "content": "Get all users active in the last 30 days.",},
}

payload := map[string]interface{}{
    "model":    "qwen2.5-coder:7b",
    "messages": messages, // Pass conversation context to chat API
    "stream":   false,
}
// ... Send HTTP POST request to /api/chat ...
```

<!-- 
SPEAKER NOTES (Hinglish):
* Go program mapping backend structure me message payloads define karein.
* Yahan index zero par system instruction define kiya hai, jo model ko strict constraints apply karne ko force karta hai.
* Payload array directly chat integration service parameter pass endpoints hit karta hai.
-->

---

# How to Read System Telemetry

<div class="columns">
<div class="col">

### Payload Response
```json
{
  "model": "qwen2.5-coder:7b",
  "message": {
    "role": "assistant",
    "content": "SELECT * FROM users WHERE last_active >= DATE_SUB(NOW(), INTERVAL 30 DAY);"
  },
  "done": true
}
```

</div>
<div class="col">

### Telemetry Check
* <span class="chip">message.content</span> **SQL Output** — The model followed the system instruction to return *only* SQL.
* <span class="chip">prompt_eval_count</span> **System Overhead** — The system prompt tokens are processed on the first turn.

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
* API execution returns aur telemetry check parameters analyse karein.
* Message assistant role content SQL syntax query return format parameters represent karta hai.
* Context evaluation parsing overhead pehli sequence run parameters compile metadata initialize state check karta hai.
-->

---

# Common Problems and How to Fix Them

<div class="callout">
  <strong>📉 System Prompt Drift</strong>
  In long conversations, the model might forget system instructions as new messages fill the context window.<br>
  <span class="fix">→ Fix: Keep the system instruction anchored at index 0 of your message payload during truncation.</span>
</div>

<div class="callout">
  <strong>🔓 Instruction Injection (Jailbreaks)</strong>
  Users can bypass system rules by writing things like <em>"Ignore all previous rules and do this."</em>.<br>
  <span class="fix">→ Fix: Wrap user inputs with clean XML tags (e.g., <code>&lt;user_query&gt;</code>) in your backend template.</span>
</div>

<div class="callout">
  <strong>⚠️ Small Model Ignorance</strong>
  Smaller models (e.g., 1.5B/3B parameters) often struggle to respect system instructions.<br>
  <span class="fix">→ Fix: Merge system prompt instructions directly into the first <code>user</code> role message.</span>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
* Production deployment guardrails system issues aur settings resolve limits evaluate karein.
* Prompt drift problem memory bounds buffer limits index parameters set logic check control rules implement karein.
* Jailbreak injection attacks prevent backend wrappers aur instruction formatting parameters setup set karein.
-->

---

# Next Up: Multi-Turn State

<div class="title-divider"></div>

### Next Up: [Module 05 · Multi-Turn State](./05-multi-turn-state.html)

* **What we will cover next:**
  * Keeping chat memory across stateless API cycles
  * Structuring and appending message history lists in Go
  * Reading context token scaling telemetry
  * Handling context overflows using sliding window buffers
* **Get ready to build interactive, state-aware chat applications!**

<!-- 
SPEAKER NOTES (Hinglish):
* System Prompts behavior controls aur parameters rules settings implementation completed.
* Lekin single-turn system parameters calls persistent session memory history represent nahi kar sakte.
* Agle module (Module 05) me hum state preservation aur conversational sliding windows history options evaluate karenge. Let's move!
-->

