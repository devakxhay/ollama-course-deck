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

# <!-- fit --> Function Calling & Tool Use

<div class="title-divider"></div>

### Connecting Local Models to External APIs

<!-- 
SPEAKER NOTES (Hinglish):
Hey guys! Welcome back. Aaj hum baat karenge ek bohot hi interesting topic ke baare me—wo hai Function Calling & Tool Use.
LLMs static hote hain, unhe real-time information (jaise current stock prices, weather details, ya DB queries) ka access nahi hota.
Is module me hum seekhenge ki kaise hum standard tools schemas define karte hain taaki local model intelligently code context me function arguments generate kar sake.
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
In dono execution structures ke workflow ko compare karte hain.
Left side par dekhiye standard Chat—yahan model query read karke normal conversational paragraphs throw kar deta hai.
Right side check karein Function Calling—yahan model standard answer dene ke bajaye ek structural JSON output return karta hai jo batata hai ki target function 'name' kya hai aur 'arguments' parameter parameters kya hain. Client application is action parameters ko consume karke dynamic code compute kar sakta hai.
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
Go client payload mapping par focus karein.
Hum request maps payload parameter structure ke andar explicit `"tools"` key specify karte hain. Yahan humne function metadata target schema setup kiya hai.
Function object structure parameters me name GetStockPrice aur parameter properties parameter define kiya hai jo batata hai ki symbol property string input required segment validation map me defined hai.
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
Model output aur telemetry values analyze karte hain.
Left column payload check karein: response body data me content section empty hai par assistant role mapping state inside tool_calls list object populate hua hai.
Right column details check karein: tool_calls flag check hotey hi frontend logic understand kar leta hai ki function trigger call select hui hai, arguments parameter property symbol me AAPL parsing successfully model sampling logic target criteria verify kar chuka hai.
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
Tool pipelines compile karte time production rules gotchas dhyan rakhein.
Pehla parameter error structural parsing failures hallucination ka hai—model random properties fill up kar deta hai, isliye arguments unmarshal filter checks backend validation layers standard follow karein.
Doosra problem looping issues hai—model tool run loops me fas jata hai, client logic loop counter limit setup karein.
Teesra engine capabilities check hai—complex tools mappings parse karne ke liye standard models ke bajaye optimized tool models target karein.
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
Chalo ab next module par chalte hain! Agle class me hum seekhenge ki kaise custom raw prompts feed karte hain aur FIM templates parsing use karke code completion engine trigger design kiya jata hai. See you in the next module!
-->

