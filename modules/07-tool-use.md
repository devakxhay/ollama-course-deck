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

<span class="module-label">Module 07 · Tool Use</span>

# <!-- fit --> <img src="../assets/ollama.png" height="60" style="vertical-align: middle; margin-right: 15px;" /> Function Calling & Tool Use

<div class="title-divider"></div>

### Connecting Local Models to External APIs

<!-- 
SPEAKER NOTES (Hinglish):
* Welcome back! Module 06 me humne raw images load karke vision-based systems build karna seekha.
* Par dynamic applications ko static data models ke aage badh kar real-time APIs aur active execution functions ki zaroorat hoti hai.
* Aaj hum seekhenge ki model constraints parameters ke sath system function declaration schemas kaise configure karein.
* Hum direct conversational replies aur structural tool calling differences compare karenge.
-->

---

# Direct Chat vs. Function Calling: What is the Difference?

<div class="columns">
<div class="col">

### Direct Chat
* **Output:** Standard conversational text responses.
* **Execution:** Direct text streaming from model parameters.
* **Limitations:** Stagnant training database cutoff limits.

</div>
<div class="col">

### Function Calling (Tool Use)
* **Output:** Structured JSON containing function names and arguments.
* **Execution:** Client runs code using the generated arguments.
* **Benefits:** Dynamically connects model to live external APIs.

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
* Direct Chat aur Function Calling pipelines ke workflow differences ko check karein.
* Left side standard chat user flow normal text format sequences return karta hai.
* Right side model standard conversation block ignore karke structured parameter keys list contain karta hai.
-->

---

# How to Declare Tools in Go

```go
payload := map[string]interface{}{
    "model":    "qwen2.5-coder:7b",
    "messages": []map[string]string{{"role": "user", "content": "What is AAPL stock price?"}},
    "tools": []map[string]interface{}{{
        "type": "function",
        "function": map[string]interface{}{
            "name":        "GetStockPrice",
            "description": "Fetch stock price",
            "parameters": map[string]interface{}{
                "type": "object",
                "properties": map[string]interface{}{
                    "symbol": map[string]string{"type": "string"},
                },
                "required": []string{"symbol"},
            },},}},}
```

<!-- 
SPEAKER NOTES (Hinglish):
* Go client payload mapping data elements check karein.
* Request JSON body properties options list me target tools metadata parameter declare kiya hai.
* Models instructions details schema rules mapping evaluate settings parameter details load karti hain.
-->

---

# How to Read Tool Call Telemetry

<div class="columns">
<div class="col">

### Telemetry Payload
```json
{
  "message": {
    "role": "assistant",
    "tool_calls": [
      {
        "function": {
          "name": "GetStockPrice",
          "arguments": {
            "symbol": "AAPL"
          }
        }
      }
    ]
  }
}
```

</div>
<div class="col">

### Telemetry Check
* <span class="chip">tool_calls</span> **Execution Flag** — If present, indicates the model opted to call a defined tool.
* <span class="chip">arguments</span> **JSON Parameters** — The arguments (e.g. AAPL) mapped directly to your function schema.

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
* Model output validation aur telemetry fields parse karein.
* Response body content check mapping index standard array object tool call data return karta hai.
* Dynamic argument values validation parsing steps application logic backend verify options checks pass karta hai.
-->

---

# Common Problems and How to Fix Them

<div class="callout">
  <strong>🧩 Schema Argument Hallucination</strong>
  Models might hallucinate values or produce invalid JSON arguments.<br>
  <span class="fix">→ Fix: Implement strict client-side JSON parsing and schema validation rules.</span>
</div>

<div class="callout">
  <strong>🔄 Tool Calling Infinite Loops</strong>
  The model may call the same tool repeatedly without returning a final response.<br>
  <span class="fix">→ Fix: Set a hard limit on maximum tool iteration cycles in the execution loop.</span>
</div>

<div class="callout">
  <strong>🚫 Model Incapability</strong>
  Standard small models frequently fail to parse complex nested tool parameters.<br>
  <span class="fix">→ Fix: Choose models explicitly optimized for tool calling (e.g., <code>qwen2.5-coder</code>).</span>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
* Tool usage production environments issues and solutions check karein.
* Arguments validation mapping check limits criteria check apply karein.
* Infinite iteration loops control karne ke liye maximum recursion limit rules register details evaluate karein.
-->

---

# Next Up: FIM & Raw Prompt Mechanics

<div class="title-divider"></div>

### Next Up: [Module 08 · Raw Prompt Mechanics](./08-raw-prompt.html)

* **What we will cover next:**
  * In-filling code sections using Fill-in-the-Middle (FIM) structures
  * Bypassing default system prompt templates using `"raw": true`
  * Injecting special boundary markers in Go templates
  * Stopping output runaways using stop token configurations
* **Get ready to build real-time code editor extensions!**

<!-- 
SPEAKER NOTES (Hinglish):
* Autonomous function calling schemas aur tool executions setup completed.
* Lekin normal API messages hamesha standard chat template structure layers bypass nahi kar sakte.
* Agle module (Module 08) me hum template engines overrides aur Fill-in-the-Middle code completion setups check karenge. Let's move!
-->

