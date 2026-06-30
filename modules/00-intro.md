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

<span class="module-label">Module 00 · Course Introduction</span>

# <!-- fit --> <img src="../assets/ollama.png" height="60" style="vertical-align: middle; margin-right: 15px;" /> Building with Ollama

<div class="title-divider"></div>

### Local LLMs in Action

<!-- 
SPEAKER NOTES (Hinglish):
* Welcome guys! "Building with Ollama: Local LLMs in Action" course me aapka swagat hai.
* Aaj hum samjhenge ki local AI ki actual demand aur benefits kya hain.
* Hum market ke different engines aur desktop client options compare karenge.
* Last me, course roadmap aur program implementation language choice discuss karenge.
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
* Sabse pehle discuss karte hain data privacy ko—aapka dynamic input data aur system prompts bilkul secure rehte hain.
* Second factor hai billing cost—token size ya api calls badhne se koi monthly billing surge nahi hota.
* Third parameter reliability aur latency hai—bina local internet network dependency ke high speed responses microsecond latency par access ho jate hain.
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
* Local AI development pipeline me custom execution choices bohot saari hain.
* Llama.cpp high-performance bare-metal engine hai par complex setups aur configurations require karta hai.
* vLLM high throughput clusters ke liye optimized hai par resource-intensive GPU requirements demand karta hai.
* LM Studio UI access ke liye theek hai par programmatic integration aur scaling support me limitation hai.
* Ollama in sabko simplify karta hai standard developer APIs aur smooth integration flow ke sath.
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
* Ollama ka development model Docker structure par based hai jo setup aur orchestration easy banata hai.
* Hum dynamic `ollama run` instructions trigger karke configurations aur weights check resolve kar sakte hain.
* Local runtime environments background api services spin up karte hain, jo HTTP endpoints expose karti hain.
* System automatically hardware check run karke workload divide karta hai CPU aur active GPUs ke beech.
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
* Hum is project code execution me Go language use karenge par structure flexible hai.
* Aap python, javascript ya kisi bhi programming language me system code structure design kar sakte hain.
* Core engine design and API specs runtime inputs me common patterns follow karte hain.
* Focus concepts par hona chahiye jaise memory handling, token parsing, and response buffers.
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
* Purane basic pipelines se lekar advance autonomous capabilities tak total 11 detailed modules plan hain.
* Hum REST calls aur payload optimization se cover karna shuru karenge.
* Next phase me streaming, JSON structures, system behaviors, memory loops, aur vision endpoints check karenge.
* Project wrap capstone local agent development aur production deployment configs ke sath hoga.
-->

---

# Let's Start This Journey!

<div class="title-divider"></div>

### Next Up: [Module 01 · REST API Internals](./01-api-basics.html)

* **What we will cover next:**
  * Endpoint payloads (`/api/generate` and `/api/chat`)
  * Reading execution telemetry & response metrics
  * Common problems and how to fix them
* **Get ready to write some code!**

<!-- 
SPEAKER NOTES (Hinglish):
* Chaliye is execution path ko practical direction dete hain.
* Agle module me hum seedhe server backend structures aur raw api payloads analyze karenge.
* Hum `/api/generate` aur `/api/chat` ke differences code benchmarks ke sath cover karenge. Let's start!
-->
