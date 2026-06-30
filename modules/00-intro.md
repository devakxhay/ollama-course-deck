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

<span class="module-label">Module 00 · Course Introduction</span>

# <!-- fit --> Building with Ollama

<div class="title-divider"></div>

### Local LLMs in Action

<!-- 
SPEAKER NOTES (Hinglish):
Hey guys! Welcome to the course "Building with Ollama: Local LLMs in Action". Aaj hum baat karenge ki hume local AI ki zaroorat kyun hai, market me iske alawa kaun se tools available hain, aur Ollama in sabse kaise behtar hai. Hum course ka roadmap aur language strategy bhi check karenge.
-->

---

# Why Local AI?

<div class="columns">
<div class="col">

### Core Benefits
* **Data Privacy:** Your data never leaves your hardware.
* **Zero Cost:** No API keys, token bills, or usage limits.
* **Offline Access:** Build and run apps without the internet.

</div>
<div class="col">

### Developer Control
* **Custom Models:** Tune parameters and system prompts directly.
* **Low Latency:** Fast local responses with zero network delay.
* **Reliability:** No downtime or API rate limits.

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
Sabse pehle samajhte hain—Why Local AI? Cloud APIs jaise OpenAI ya Claude bohot badhiya hain, par production systems me local AI ke teen bade fayde hain:
Pehla hai privacy—aapka proprietary data server se bahar nahi jata.
Doosra hai cost—yahan koi per-token bill ya subscription nahi hai. Unlimited runs completely free hain.
Teesra hai reliability aur latency—bina internet ke bhi microsecond latency par local models response generate karte hain bina kisi rate limits ke.
-->

---

# The Local AI Tool Landscape

<div class="columns">
<div class="col">

### Raw Engines
* **Llama.cpp:** The core C++ engine. Fast but complex to set up.
* **vLLM:** Great for server hosting. High GPU demand.

</div>
<div class="col">

### Desktop Apps & APIs
* **LM Studio:** Nice UI, but hard to integrate into code.
* **Ollama:** Simple, Docker-like flow, with built-in REST APIs.

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
Local LLMs chalane ke liye market me kayi saare tools hain.
Llama.cpp bilkul bare-metal engine hai, jo bohot fast hai par configurations bohot hard hain.
vLLM production servers ke liye best hai par uske liye heavy GPU stack chahiye hota hai.
LM Studio jaise tools me badhiya GUI milta hai but code integration aur deployment workflow thoda clumsy ho jata hai.
Aur phir aata hai Ollama—jo dono worlds ke best parts ko blend karta hai.
-->

---

# Why Ollama?

<div class="columns">
<div class="col">

### Docker-like Experience
* **Single Command Run:** Download and start models using `ollama run`.
* **Standard API:** Exposes simple HTTP endpoints automatically.
* **Modelfiles:** Define custom settings in a single config file.

</div>
<div class="col">

### System Friendly
* **Resource Management:** Runs as a background service.
* **Auto CPU/GPU Switch:** Runs on CPU if no GPU is found.
* **Cross Platform:** Works on macOS, Windows, and Linux.

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
Ollama ka design bilkul Docker ki tarah hai. Ek single command se model pull aur run ho jata hai.
Aapko raw weights ya configuration layers manage karne ki zaroorat nahi padti.
Background me ye ek lightweight API service chalta hai jo JSON format me generate aur chat endpoints expose karta hai.
Sabse best part ye hai ki ye automatic resource balancing karta hai—agar GPU available hai toh GPU use karega, nahi toh CPU par fall back ho jayega.
-->

---

# Why Go? (And Why Language Doesn't Matter)

<div class="columns">
<div class="col">

### Language Choice: Go
* **High Performance:** Fast startup, small memory footprint.
* **Concurrency:** Great for multiple user chat queues.
* **No Syntax Focus:** We will keep syntax simple and clean.

</div>
<div class="col">

### For Other Developers
* **Python/JS/Java:** Do not worry, concepts are the same.
* **Core Logic:** Focus on the concepts, not the language.
* **Translation:** Code patterns translate to any language.

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
Is course me hum apna Ollama project Go language ka use karke banayenge. Lekin agar aap Python, JavaScript, ya Java developer hain, toh bilkul tension mat lijiye aur tab close mat kariye!
AI ke is era me programming language sirf syntax noise hai. Agar hum syntax par zyada focus karenge, toh main concepts miss kar denge. 
Hum language ko completely ignore nahi karenge—agar koi Go-specific issue ya type mismatch aayega toh hum use jaldi se fix karke aage badhenge. Lekin apni core energy language par waste mat kijiye.
Focus kijiye concepts par: context kaise manage karna hai, streaming kaise handle karni hai, aur AI apps kaise architect karni hain. Jo logic aap yahan seekhenge, wo aapki workspace language me perfect translate hoga.
-->

---

# What We Will Cover

<div class="columns">
<div class="col">

* API Basics (Generate vs. Chat)
* Calling APIs from Code
* Streaming Responses (Latency Fix)
* Structured Output (JSON Mode)
* System Prompts & Multi-Turn State

</div>
<div class="col">

* Vision Input (Multimodal Prompts)
* Function Calling & Tool Use
* FIM & Raw Prompt Mechanics
* Embeddings & Semantic Search
* Capstone (Building a Local Agent)

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
Is series me hum total 10 episodes cover karenge—bilkul zero fluff aur straight to capability. 
Pehle hum API basics aur backend languages se calling seekhenge. Phir streaming aur JSON structured output par jump karenge.
Aage chal kar hum system prompts, vision input, function calling, FIM (Fill-in-the-Middle) code completion aur embeddings cover karenge.
Last me hum ek full-blown capstone mini local agent build karenge jo in saare concepts ko aapas me integrate karega.
-->

---

# Let's Start This Journey!

<div class="title-divider"></div>

### Next Up: [Module 01 · REST API Internals](./01-api-basics.md)

* **What we will cover next:**
  * Endpoint payloads (`/api/generate` and `/api/chat`)
  * Reading execution telemetry & response metrics
  * Production gotchas, timeouts, and parallel queues
* **Get ready to write some code!**

<!-- 
SPEAKER NOTES (Hinglish):
Toh chaliye guys, is journey ko shuru karte hain! Agle module me hum seedha REST API Internals par jump karenge. Hum API endpoints, payloads, latency metrics, aur production setups ko live test karenge. Let's get started!
-->
