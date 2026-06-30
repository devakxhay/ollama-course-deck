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

<span class="module-label">Module 03 · Structured Output (JSON Mode)</span>

# <!-- fit --> Structured Output

<div class="title-divider"></div>

### Enforcing Schema Conformity with JSON Mode

<!-- 
SPEAKER NOTES (Hinglish):
Hey guys! Welcome back. Aaj hum baat karenge local application integration ke ek bohot important pillars ke baare me—wo hai Structured Output.
Jab hum raw LLMs use karte hain, toh wo random text, conversational explanations, ya formatting errors return karte hain jise hamare backend systems direct parse nahi kar pate.
Is session me hum dekhenge ki kaise Ollama ke built-in JSON Mode ko leverage karke hum system-level valid schemas produce kara sakte hain.
-->

---

# Raw Text vs. JSON Mode: What is the Difference?

<div class="columns">
<div class="col">

### Raw Text Output
* **Response Format:** Plain string, markdown blocks, conversational preamble.
* **Validation:** Custom regex or LLM post-processing required.
* **Failure Modes:** Preambles like *"Here is your JSON"* corrupt programmatic parsing.

</div>
<div class="col">

### JSON Mode (`format: "json"`)
* **Response Format:** Guarantees syntactically valid JSON.
* **Validation:** Enforced directly during token sampling.
* **Failure Modes:** Might hit token limits if the model loops trying to balance brackets.

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
In dono patterns me difference samajhna zaroori hai.
Left side par dekhein: jab hum normal request bhejte hain, toh model conversational context generate karta hai jaise "Here is the details you requested" aur codeblocks me JSON wrap kar deta hai. Ye standard parser ko crash kar deta hai.
Right side par check karein: Ollama me format: "json" config pass karne par, model ka sampling layer target tokens ko constrain karta hai taaki direct, raw, and pure JSON output generate ho bina kisi text wrapper ke.
-->

---

# How to Request JSON Output in Go

```go
// 1. Request JSON mode in your payload
payload := map[string]interface{}{
    "model":  "qwen2.5-coder:7b",
    "prompt": "Analyze: 'Book flight to Delhi'. Return JSON with query, intent, tags.",
    "format": "json", // Enforces valid JSON generation
    "stream": false,
}
// ... Send HTTP request to /api/generate ...
var data struct {
    Query  string   `json:"query"`
    Intent string   `json:"intent"`
    Tags   []string `json:"tags"`
}
json.Unmarshal([]byte(apiResponse.Response), &data)
```

<!-- 
SPEAKER NOTES (Hinglish):
Chalo iska backend pipeline implementation check karte hain.
Sabse pehle hum local target schema setup karte hain struct mapping define karke.
Request call ke execution payload body me notice karein: hume prompt me clean metadata instructions ke sath explicit format: "json" set karna padta hai.
HTTP response capture hone par, response key ke andar pure string value me structured output dynamic JSON text return hota hai, jise hum directly target struct fields me parse kar sakte hain.
-->

---

# How to Read Structured Telemetry

<div class="columns">
<div class="col">

### Payload Response
```json
{
  "model": "qwen2.5-coder:7b",
  "response": "{\n  \"query\": \"Book flight to Delhi\",\n  \"intent\": \"book_flight\",\n  \"tags\": [\"travel\", \"flight\", \"delhi\"]\n}",
  "done": true
}
```

</div>
<div class="col">

### Telemetry Check
* <span class="chip">response</span> **JSON String** — Stringified JSON escape sequence inside response body.
* <span class="chip">format: "json"</span> **Constraint Output** — Sampling engine constraints model outputs to open/close brackets.

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
Inference call ke response JSON payload mapping ko deepdive karein.
Left side par dekhiye: response raw string structure key-value map format me structured data content dynamically represent karta hai.
Ollama system backend par compiler schema constraint apply karta hai. System dynamic logic layers control karta hai taaki open-braces and close-braces perfectly close and maintain ho sake.
-->

---

# Common Problems and How to Fix Them

<div class="callout">
  <strong>💬 Prompt Must contain 'JSON'</strong>
  Ollama JSON mode fails or freezes if the prompt does not contain the word "JSON".<br>
  <span class="fix">→ Fix: Always explicitly specify <em>"Output in JSON format"</em> in your prompt or system instructions.</span>
</div>

<div class="callout">
  <strong>🧩 Schema Validation vs. Syntactic Validation</strong>
  JSON mode only guarantees valid JSON syntax, NOT that the keys and types are correct.<br>
  <span class="fix">→ Fix: Validate keys client-side or use a schema validation framework like JSON Schema or Pydantic.</span>
</div>

<div class="callout">
  <strong>⏳ Latency and Stuck Generations</strong>
  Models can loop indefinitely trying to format large JSON structures, hitting timeouts.<br>
  <span class="fix">→ Fix: Set tight <code>num_predict</code> constraints and avoid deeply nested array formats.</span>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
Structured outputs use karte time production environment me in gotchas ka dhyan rakhein.
Sabse pehle, agar aapne input parameters me format to json set kiya hai but prompt me word JSON explicit include nahi kiya toh model runtime parsing par crash ya freeze ho sakta hai. Isliye instructions me "JSON" compile keyword standard practice banalein.
Doosra, standard validation levels syntax validity tak restricted hai, schema structural checks (keys correctness) verify karne ke liye client side par validation filter execute karein.
Teesra, model infinite structures formatting loops me na fas jaye, isliye output sizes control constraints configuration standard options me set karein.
-->

---

# Next Up: System Prompts

<div class="title-divider"></div>

### Next Up: [Module 04 · System Prompts](./04-system-prompts.md)

* **What we will cover next:**
  * Setting identity and tone constraints via `role: system`
  * Differentiating System role behavior from ad-hoc User queries
  * Implementing system-level guardrails in Go
  * Mitigating prompt drift and injection attacks in production
* **Get ready to command assistant behaviors structurally!**

<!-- 
SPEAKER NOTES (Hinglish):
Chalo ab next module par chalte hain! Agle class me hum seekhenge ki kaise system level custom roles define kiye jate hain aur application ke boundaries rules configure kiye jate hain. See you in the next module!
-->

