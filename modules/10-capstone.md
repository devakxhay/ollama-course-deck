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
Hey guys! Welcome to the Capstone module. Aaj hum baat karenge local systems me agent orchestration ke baare me—yaani humne pichle 9 modules me jo bhi seekha hai, un sabhi building blocks ko ek sath jodkar ek autonomous local agent build karenge.
Hum is module me visual schemas, memory history, RAG vectors, aur tool invocation pipelines ka convergence dekhenge conceptual level par.
Hum is slide deck me koi dynamic coding snippets nahi dikhayenge, balki pure system design aur working logic par focus karenge.
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
Chalo is Cognitive loop yaani Reasoning and Acting structure ko samajhte hain.
Left column me process dekhiye: user query aate hi model directly answer nahi karta. Wo pehle sochna shuru karta hai, ek dynamic plan banata hai, suitable tool call (arguments ke sath) choose karta hai, and client-side code se execute kara kar observations capture karta hai.
Right side flow structure dekhiye: ye loop tab tak chalta hai jab tak model satisfy nahi ho jata ki correct target data source retrieve ho gaya hai, fir target response user ko deliver ho jata hai.
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
RAG aur active tools ke integration workflow ko check karte hain.
Left column me dekhiye RAG pipeline—jaise hi request aati hai, orchestrator nomic-embed model use karke semantic search call execute karta hai aur relevant file segments pull karke system prompt buffer inside inject kar deta hai.
Right column check karein tool loop—agar model lagta hai ki computational query ke liye external logic execute karni hai, toh wo system tool call parameter throw karta hai. Backend library commands run karke system observation state loop update kar deti hai.
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
Agent parameters me processing telemetry details check karte hain.
Left column check karein: har single reasoning loop iteration me token payload exponentially badhta hai kyunki pichle thoughts aur observations system memory stack inside combine ho jate hain.
Right column check karein prompt_eval_count: yahan parsing latency high dikhegi kyunki model ko updates trace karne ke liye historical actions and tool outputs re-evaluate karne padte hain.
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
GTX 1650 hardware boundaries par agent deploy karte time key optimization rules follow karein.
Pehla issue 4GB VRAM cap ka hai—chat and embed models simultaneously load hone par VRAM thrashing speed issues aate hain. Iske liye 1.5B/3B small parameters and quantized models use karein aur OLLAMA_NUM_PARALLEL standard limits to 1 fix rakhein.
Doosra bottleneck infinite loops ka hai—model stuck ho sakta hai, client orchestrator side par loops count cap counters validate karein.
Teesra VRAM context size issue hai—long strings CPU processing limits fall ho sakti hain, isliye context limit strictly 4k limit set parameters block karein.
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
Chalo ab final module par chalte hain! Agle class me hum seekhenge ki kaise in local models ko deploy kiya jata hai scaling architectures use karke aur standard load balancing mechanisms trigger setup karte hain. See you in the next module!
-->

