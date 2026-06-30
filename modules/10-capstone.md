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

<span class="module-label">Module 10 · Capstone Project</span>

# <!-- fit --> <img src="../assets/ollama.png" height="60" style="vertical-align: middle; margin-right: 15px;" /> Building a Local Agent

<div class="title-divider"></div>

### Orchestrating Cognitive Loops on Local Hardware

<!-- 
SPEAKER NOTES (Hinglish):
* Welcome back! Module 09 me humne context search queries resolve karne ke liye embeddings create kiye.
* Aaj hum in saare modules ke dynamic building blocks ko merge karke ek full autonomous local agent construct karenge.
* Hum visual schemas, memory histories, vector retrieves, aur tool setups ka system design convergence check karenge.
* Is conceptual presentation me hum standard logic patterns aur orchestrator loops structure par detail me focus karenge.
-->

---

# The ReAct Loop: How Local Agents Think and Act

<div class="columns">
<div class="col">

### Step-by-Step Cognitive Cycle
1. **User Goal:** Defines the primary objective.
2. **Thought:** Agent reasons about what to do next.
3. **Action:** Agent selects a tool and generates arguments.
4. **Observation:** Client executes the tool and returns results.
5. **Looping:** Process repeats until goal is accomplished.

</div>
<div class="col">

### Architectural Flow
```
   [User Input]
        │
        ▼
   ┌────┴────────┐
   │ Thought     │ ◄───┐
   └────┬────────┘     │
        ▼              │
   ┌────┴────────┐     │ [Observation]
   │ Action (Tool)│    │
   └────┬────────┘     │
        ▼              │
   ┌────┴────────┐     │
   │ Exec Code   ├─────┘
   └────┬────────┘
        ▼
   [Final Answer]
```

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
* ReAct cognitive reasoning loop execution cycle check karein.
* Left side steps check karein: user goal assign hotey hi model execution sequence me thoughts process logic determine karta hai.
* Right side flow diagram verify karein: client observations dynamically loop me return hokar model next choices update karti hain.
-->

---

# Semantic Search and Tool Orchestration Workflow

<div class="columns">
<div class="col">

### Knowledge Base Integration (RAG)
* **Lookup Phase:** The orchestrator converts user query to vectors.
* **Context Loading:** Fetches matches from local vector database.
* **Instruction Feed:** Context is injected as background facts.

</div>
<div class="col">

### Active Tool Execution
* **Decider Phase:** Model checks if it needs external APIs (e.g., Weather, Files).
* **Code Execution:** Application runs the local system command.
* **Feedback Loop:** Result is fed back to prompt memory.

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
* RAG system features aur tool integration orchestrations check karein.
* Left column check retrieve process: vector representations similarity inputs parse karke context buffer update kiya jata hai.
* Right column actions process: dynamic conditions match database commands execution observations memory index inject karti hain.
-->

---

# How to Read Agent Execution Telemetry

<div class="columns">
<div class="col">

### Turn Slicing Loop
* **Iteration 1:** Input size: 500 tokens (System + User).
* **Iteration 2:** Input size: 850 tokens (History + Tool Schema + Call 1 + Observation 1).
* **Iteration 3:** Input size: 1200 tokens (Cumulative execution logs).

</div>
<div class="col">

### Telemetry Check
* <span class="chip">prompt_eval_count</span> **Loop Inflation** — Each iteration re-evaluates all previous tool calls and results.
* <span class="chip">eval_count</span> **Decisions** — Tokens generated by the model to state its reasoning.

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
* Loop iterations memory telemetry parameters details study karein.
* Left side token calculations turn progression limits linear scaling state complexity trace check karein.
* Right side prompt_eval_count limits evaluation loops duration memory caching effects parameters analyze karein.
-->

---

# Common Problems and How to Fix Them

<div class="callout">
  <strong>📉 GTX 1650 4GB VRAM Blowouts</strong>
  Running chat and embed models concurrently triggers VRAM limits, causing model thrashing.<br>
  <span class="fix">→ Fix: Use 1.5B/3B models. Set <code>OLLAMA_NUM_PARALLEL=1</code>.</span>
</div>

<div class="callout">
  <strong>⏳ Infinite Tool Call Loops</strong>
  Underpowered local models can get stuck calling the same tool repeatedly.<br>
  <span class="fix">→ Fix: Set a strict <code>max_iterations = 4</code> counter in your orchestrator.</span>
</div>

<div class="callout">
  <strong>⚠️ Prompt Evaluation Latency</strong>
  Processing contexts beyond 4k tokens leads to CPU fallbacks and severe lag.<br>
  <span class="fix">→ Fix: Restrict <code>num_ctx: 4096</code> and truncate reasoning history.</span>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
* GTX 1650 memory capacity constraints aur runtime gotchas solutions verify karein.
* VRAM failures prevent karne ke liye quantization model parameters select default value setup range check karein.
* Infinite reasoning loop limits checks and context size memory windows restrictions backend rules settings.
-->

---

# Next Up: Local LLMs in Production

<div class="title-divider"></div>

### Next Up: [Module 11 · Course Outro](./11-end.html)

* **What we will cover next:**
  * Scaling configurations from sandbox machines to HA cluster architectures
  * Designing multi-node request balancing topologies using load balancers
  * Monitoring active token throughput, latency profiles, and VRAM utilization
  * Resolving queue congestion, container loading lag, and GPU OOM errors
* **Get ready to deploy your LLMs to production!**

<!-- 
SPEAKER NOTES (Hinglish):
* Local autonomous ReAct agent design system structures evaluation complete target evaluate kiya.
* Lekin local setup prototype se production cluster setup options scale parameters shift dynamics critical hotey hain.
* Agle module (Module 11) me hum scaling configurations, containerized nodes, load balancing aur server queues resolve setup check karenge. Let's start!
-->

