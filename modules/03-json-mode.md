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

<span class="module-label">Module 03 · Structured Output (JSON Mode)</span>

# <!-- fit --> <img src="../assets/ollama.png" height="60" style="vertical-align: middle; margin-right: 15px;" /> Structured Output

<div class="title-divider"></div>

### Enforcing Schema Conformity with JSON Mode

<!-- 
SPEAKER NOTES (Hinglish):
* Welcome back! Module 02 me humne perceived latency drop karne ke liye dynamic streaming check kiya.
* Par backend services aur production pipelines me plain text outputs parse karna bohot unpredictable ho sakta hai.
* Aaj hum seekhenge ki kaise structure parsing failure errors ko solve karne ke liye Ollama ka JSON Mode configure kiya jata hai.
* Hum raw textual results aur schema-constrained outputs ke difference aur implementations ko check karenge.
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
* Raw Text Output aur schema-enforced JSON Mode ka primary difference analyze karein.
* Left side par normal generation flow dynamic conversational text return karta hai, jo system configurations crash kar sakta hai.
* Right side par constraints schema apply hone se compiler parsing backend parameters JSON syntax compliance restrict kar dete hain.
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
* Program pipelines implementation setups Go targets define karein.
* Payload creation options array configuration check parameters me format JSON specify kiya jata hai.
* HTTP response payload target structs me unmarshal karke key parameters value verification levels pass kiye jate hain.
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
* System output response return formatting payload target evaluation parameter check karein.
* Left column data metrics string representation return structures dynamic values contain karta hai.
* System backend parameter sampling steps verify validation ensure karte hain brackets open/close status mapping rules resolve karne ke liye.
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
* JSON generation pipeline gotchas aur constraints solutions trace details check karein.
* Prompt input details compile levels me "JSON" keyword use karna strict mandatory protocol hai server side freezing block avoid karne ke liye.
* Output size parameter constraints validation verify settings local client checks rules filter apply karein.
-->

---

# Next Up: System Prompts

<div class="title-divider"></div>

### Next Up: [Module 04 · System Prompts](./04-system-prompts.html)

* **What we will cover next:**
  * Setting identity and tone constraints via `role: system`
  * Differentiating System role behavior from ad-hoc User queries
  * Implementing system-level guardrails in Go
  * Mitigating prompt drift and injection attacks in production
* **Get ready to command assistant behaviors structurally!**

<!-- 
SPEAKER NOTES (Hinglish):
* JSON Mode structured data delivery aur execution validation setup module complete hota hai.
* Lekin simple structured queries parameters direct system limits control configurations settings handle nahi kar sakti.
* Agle module (Module 04) me hum system-level rules define karne ke liye System Prompts configurations aur constraints methods check karenge. Let's start!
-->

