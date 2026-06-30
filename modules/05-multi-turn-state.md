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

# <!-- fit --> <img src="../assets/ollama.png" height="60" style="vertical-align: middle; margin-right: 15px;" /> Multi-Turn State

<div class="title-divider"></div>

### Maintaining Conversation History and Memory

<!-- 
SPEAKER NOTES (Hinglish):
* Welcome back! Module 04 me humne system instructions aur behaviors setup standard guidelines check kiye.
* Par real chatbots me persistent memory ke bina back-and-forth dynamic conversations maintain nahi ki ja sakti.
* Aaj hum seekhenge ki stateless APIs ke upar dynamic conversation history queue aur database levels par kaise manage karein.
* Hum stateless request blocks aur stateful history array parameters ke difference evaluate karenge.
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
* Stateless API aur Stateful history ke mechanisms ko compare karte hain.
* Stateless mode me server request complete hotey hi session clean database entries block clear kar deta hai.
* Stateful structure me memory history lists maintain ki jati hain aur client har request call par poori history array pass karta hai.
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
* Go program structural logic database setups ko observe karein.
* Humne dynamic message memory slices block define kiye hain parameters manage karne ke liye.
* New messages list me append hote hain, aur assistant confirmation tokens successfully append parameters register karte hain.
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
* Session history token limits growth mathematical steps trace karein.
* Turn 1 flat inputs run hote hain par Turn 2 start hote hi old conversations arrays context add compute target bounds increase karte hain.
* linear progression analysis index elements scale verify control rules check karta hai.
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
* Tokens progression logic aur execution complex parameters calculations analyze karein.
* Sum totals calculation turn metrics linear progress complexity representation parameters control map trace.
* Stateless servers prompt evaluation latency overhead dynamically increase parameters process flow details.
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
* Memory history structures optimization rule limits parameters checks handle karein.
* Sliding window array implementations use karke first indexing system instructions retain settings configure target.
* State failure, cancellation exceptions aur transaction controls backend queue status check methods.
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
* Stateless history memory updates and multi-turn loops implementation completed.
* Lekin text-only chats real-world media assets data segments parse karne me fail ho jati hain.
* Agle module (Module 06) me hum image processing aur multimodal vision structures application details check karenge. Let's move!
-->

