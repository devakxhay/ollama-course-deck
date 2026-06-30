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

<span class="module-label">Module 05 · Multi-Turn State</span>

# <!-- fit --> Multi-Turn State

<div class="title-divider"></div>

### Maintaining Conversation History and Memory

<!-- 
SPEAKER NOTES (Hinglish):
Hey guys! Welcome back. Aaj hum baat karenge ek bohot hi important production requirement ke baare me—wo hai Multi-Turn State.
LLMs architecture wise completely stateless hote hain. Wo ye yaad nahi rakh sakte ki pichle query me user ne kya poocha tha.
Is session me hum seekhenge ki kaise application logic layer par conversation history array maintain karke hum dynamic memory features build kar sakte hain.
-->

---

# Stateless API vs. Stateful History: What is the Difference?

<div class="columns">
<div class="col">

### Stateless API
* **Behavior:** The model processes each HTTP request independently.
* **Context:** No memory of past requests exists inside the server.
* **Overhead:** Low server-side storage; simple requests.

</div>
<div class="col">

### Stateful History
* **Behavior:** The application maintains and updates a message queue.
* **Context:** The full historical array is sent with every new request.
* **Overhead:** Higher token payloads and increasing prompt evaluation costs.

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
Chalo iske primary differences samajhte hain.
Left side par dekhein Stateless API: Ollama by default request bypass hone ke baad session clean kar deta hai. Har request fresh start hoti hai.
Right side par Stateful History: yahan client ya backend server side par local db ya array list me data accumulate hota rehta hai, aur pure conversation timeline array ko har new hit ke sath send kiya jata hai taaki chat context maintained rahe.
-->

---

# How to Maintain Conversation History in Go

```go
type Message struct {
    Role    string `json:"role"`
    Content string `json:"content"`
}

// 1. Maintain a slice of historical messages in memory/db
var history []Message

func ChatTurn(userQuery string) string {
    // Append the new user message to history
    history = append(history, Message{Role: "user", Content: userQuery})
    
    // ... Send complete 'history' slice to /api/chat ...
    
    // Append the assistant's reply to maintain state
    history = append(history, Message{Role: "assistant", Content: reply.Message.Content})
    return reply.Message.Content
}
```

<!-- 
SPEAKER NOTES (Hinglish):
Go code logic par focus karte hain.
Humne yahan ek dynamic struct list pointer use kiya hai.
User call aate hi hum history list me type user aur string append karte hain, is complete history object slice array ko hum API body payload JSON format me encode karke query send karte hain. Response returning phase par assistant response stream body ko array index state me store kar dete hain.
-->

---

# How to Read Context Telemetry

<div class="columns">
<div class="col">

### Sequence Evaluation
* **Turn 1:** User prompt: 10 tokens. Model eval: 10 tokens.
* **Turn 2:** History (10+20) + User (10) = 40 input tokens.
* **Turn 3:** History (40+30) + User (15) = 85 input tokens.

</div>
<div class="col">

### Telemetry Check
* <span class="chip">prompt_eval_count</span> **Linear Growth** — Increases on each turn as old tokens are re-evaluated.
* <span class="chip">eval_count</span> **Generation Output** — Remains dependent only on the length of the new response.

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
Chalo history size metrics ke context calculation ko deepdive karte hain.
Left column me turn mapping sequence check karein:
Pehle, Turn 1 me user prompt 10 tokens ka hai, toh engine sirf wahi 10 tokens evaluate karta hai. Suppose model ne 20 tokens ka reply generate kiya.
Ab Turn 2 me, hamara history segment (10 user + 20 assistant = 30 tokens) aur current user query (10 tokens) combine hokar total input 40 tokens banate hain. Isliye prompt_eval_count 40 input tokens par update ho jata hai. Let's say model ne 30 tokens ka reply diya.
Turn 3 me, pichli complete chain (10+20+10+30 = 70 tokens) aur new user prompt (15 tokens) add hokar 85 input tokens banate hain.
Kyunki model architecture stateless hai, har turn par server ko system validation ke liye poori history read karni padti hai, jisse input size linearly scale hota hai. Right column me eval_count wahi tokens output dikhata hai jo current session response generation me lagte hain.
-->

---

# Why Context Growth is Linear

<div class="columns">
<div class="col">

### The Token Progression Formula
Let $U_i$ be user query tokens and $A_i$ be assistant response tokens at turn $i$:

$$\text{Input Payload}_N = \sum_{i=1}^{N-1} (U_i + A_i) + U_N$$

If we assume a constant average user size ($u$) and assistant size ($a$):

$$\text{Payload}_N \approx N \cdot (u + a) - a$$

</div>
<div class="col">

### Architectural Cost
* **Stateless Execution:** Ollama does not maintain chat states in memory. Every HTTP request starts compilation from scratch.
* **Token Complexity:** The input size scales at **$\mathcal{O}(N)$ linear complexity**.
* **TTFB Cost:** Prompt evaluation duration grows longer on every turn as the history expands.

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
Chalo is linear growth ke peeche ki simple mathematics samajhte hain.
Left side par dekhiye token progression formula—har request payload me pichle saare turns ka dynamic sum send hota hai plus current user request size. Agar user aur assistant response size average scale par maintain rahein toh payload growth N times grow karti hai.
Right side par check karein architectural cost—kyunki state server par save nahi hoti, toh Ollama ko linear complexity O(N) me context evaluate karna padta hai. Iska side effect ye hai ki chat history badi hone par prompt evaluation duration aur TTFB scale ho jate hain.
-->

---

# Common Problems and How to Fix Them

<div class="callout">
  <strong>📏 Context Window Overflow</strong>
  Default limit is 2k tokens. Exceeding it discards early messages.<br>
  <span class="fix">→ Fix: Implement a sliding window buffer. Keep system prompt at index 0.</span>
</div>

<div class="callout">
  <strong>⏳ Time Taken for First Token/Byte (TTFB) Inflation in Long Chats</strong>
  Re-evaluating long history increases TTFB latency.<br>
  <span class="fix">→ Fix: Expand context window using <code>"num_ctx": 8192</code>.</span>
</div>

<div class="callout">
  <strong>🔄 Diverged History State</strong>
  Canceled generations or network drops corrupt history.<br>
  <span class="fix">→ Fix: Append to memory only after response is fully received.</span>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
Multi-turn operations me production scalability issues ko handle karne ke tips dekhiye.
Pehla bottleneck context limit limits limit reach hone par aata hai, iske liye array windowing strategy use karein—purane chat index arrays block karein par index zero system configuration rules message maintain rakhein.
Doosra problem high parsing TTFB values ka hai, options configurations options me num_ctx context parameters expand karke load balance check karein.
Teesra issue network disconnect case me state mismatch ka hai, transaction locks validation checks response validation complete hone par hi memory stack update karein.
-->

---

# Next Up: Multimodal Vision Input

<div class="title-divider"></div>

### Next Up: [Module 06 · Multimodal Vision](./06-vision-input.html)

* **What we will cover next:**
  * Enabling multimodal capabilities with vision models
  * Encoding local image files into base64 strings in Go
  * Understanding image-token grid mapping projection metrics
  * Solving VRAM constraints and network payload bottlenecks
* **Get ready to build sight-enabled AI systems!**

<!-- 
SPEAKER NOTES (Hinglish):
Chalo ab next module par chalte hain! Agle class me hum seekhenge ki kaise images ko process karke context window structure extend karte hain aur local multimodal applications build karte hain. See you in the next module!
-->

