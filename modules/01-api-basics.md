---
marp: true
theme: default
paginate: true
header: 'Ollama in Your Project | Module 01'
footer: 'AI in Production Course'
style: |
  @import "../templates/theme.css";
  section {
    background: var(--color-bg);
    color: var(--color-text);
  }
  h1, h2, h3 {
    color: var(--color-accent);
  }
  footer, header {
    color: var(--color-muted);
  }
  .columns {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 1rem;
  }
---

# Ollama API Architecture
### Master Module 01: `/api/generate` vs `/api/chat`

<!-- 
SPEAKER NOTES (Hinglish):
Hello engineers! Agar aap local LLMs ko apne full-stack core backend controllers me integrate kar rahe ho, toh standard desktop clients ya WebUI ka use karna band karo. 
Ollama background me ek persistent daemon architecture ke upar run hota hai, aur use code se target karne ke liye hamare paas do foundational REST endpoints hain: /api/generate aur /api/chat. 
Aaj hum in dono pipelines ke core differences aur execution mechanics ko line-by-line breakdown karenge.
-->

---

# The Core Architectural Division

<div class="columns">
<div>

### /api/generate
* **Execution Pattern:** Single-turn atomic completion.
* **Input Layer:** Flat prompt string.
* **State Management:** Fully stateless transaction.
* **Best Used For:** Automated parsing, strict text classification, code snippet writing.
</div>

<div>

### /api/chat
* **Execution Pattern:** Stateful multi-turn loops.
* **Input Layer:** Structured structural message arrays.
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

# Payload Blueprint: `/api/generate`

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

# Payload Blueprint: `/api/chat`

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
* **VRAM Load Time (`load_duration`):** Cold start overhead when model is not active in VRAM.
* **Prompt Processing Speed:** Calculated as `(prompt_eval_count / prompt_eval_duration) * 1e9` tokens/sec.
* **Generation Throughput (`eval_count`):** Actual inference speed of the model.

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

* **VRAM Eviction Timeout:** Ollama unloads models from VRAM after 5 minutes of inactivity (`OLLAMA_KEEP_ALIVE=5m`).
  * *Resolution:* Set `OLLAMA_KEEP_ALIVE=-1` in environment variables to keep the model loaded indefinitely.
* **Context Length Limits:** Default context is often 2048 tokens. Long chat histories get truncated silently.
  * *Resolution:* Explicitly set `"num_ctx": 8192` inside the `options` block of your payload.
* **Concurrency Bottleneck:** Single worker queue defaults to serial execution.
  * *Resolution:* Configure `OLLAMA_NUM_PARALLEL=4` to allow concurrent stream processing.

<!-- 
SPEAKER NOTES (Hinglish):
Production deployment me ye gotchas aapka experience kharab kar sakte hain. 
Default behavior me 5 mins inactive rehne par model memory se offload ho jata hai, jisse next request par heavy cold start loading overhead face karna padta hai.
Ise fix karne ke liye environment me OLLAMA_KEEP_ALIVE ko minus one set karo.
Context window support defaults to 2048 tokens, large payloads and history preserve karne ke liye options object me num_ctx parameter manually provide karna zaroori hai.
Concurrency handling ke liye, hardware resource availability ke according OLLAMA_NUM_PARALLEL set karna zaroori hai.
-->