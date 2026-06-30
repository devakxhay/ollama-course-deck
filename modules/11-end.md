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

<span class="module-label">Module 11 · Course Outro</span>

# <!-- fit --> <img src="../assets/ollama.png" height="60" style="vertical-align: middle; margin-right: 15px;" /> Local LLMs in Production

<div class="title-divider"></div>

### Scalability, Optimization, and Future Steps

<!-- 
SPEAKER NOTES (Hinglish):
* Welcome back! Module 10 me humne cognitive ReAct loops orchestrate karke local agent system design compare kiya.
* Aaj is final module me hum local sandbox parameters se production scaling methods explore karenge.
* Hum scale-out architectures, request load balancing, concurrency, aur scheduling configurations check karenge.
* Chalo local development runtime configurations se production clusters deployment steps verify karte hain.
-->

---

# Local Instance vs. Production Cluster: What is the Difference?

<div class="columns">
<div class="col">

### Local Sandbox Instance
* **Concurreny:** Serves a single developer or client connection at a time.
* **VRAM Limits:** Bound to a single local GPU (e.g. GTX 1650 4GB).
* **Serving:** Manual startup and model loading cycles.

</div>
<div class="col">

### Production Serving Cluster
* **Concurreny:** Scales to handle thousands of concurrent API requests.
* **VRAM Limits:** Distributed nodes with VRAM caching pools.
* **Serving:** Auto-scaled, containerized microservices (Docker/K8s).

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
* Local sandboxes aur scale clusters setups me key difference check karein.
* Left side local instance setups single user queue runtime limits standard par restrict rakhte hain.
* Right side production servers dynamic microservices nodes, resource pools and scale balancing verify karte hain.
-->

---

# How to Scale Local Ollama Instances

<div class="columns">
<div class="col">

### Scaling Parameters
* **Load Balancing:** Use NGINX/HAProxy to route requests to multiple GPU endpoints.
* **Persistent Cache:** Set `OLLAMA_KEEP_ALIVE=-1` to keep models active in VRAM.
* **Worker Pools:** Map queries to worker pipelines to balance GPU threads.

</div>
<div class="col">

### Cluster Topology
```
      [Client Request Loop]
               │
               ▼
     ┌─────────┴─────────┐
     │   Load Balancer   │
     └─────────┬─────────┘
        ┌──────┼──────┐
        ▼      ▼      ▼
     ┌──┴──┐┌──┴──┐┌──┴──┐
     │Node1││Node2││Node3│
     └─────┘└─────┘└─────┘
```

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
* Multiple GPU instances scale-out options evaluate karein.
* Left side load balancing parameters parameters set aur keep_alive keep parameter minus one logic set verify.
* Right side topology setups request execution nodes divide parameters load distribution criteria test karte hain.
-->

---

# How to Monitor Production Telemetry

<div class="columns">
<div class="col">

### Performance Target Metrics
* **Token Throughput:** Tracks tokens generated per second to monitor speed.
* **Inference Queue Length:** Monitors request backlog on GPU workers.
* **Resource Saturation:** Evaluates VRAM utilization and temperature levels.

</div>
<div class="col">

### Telemetry Check
* <span class="chip">Throughput</span> **Tokens/Sec** — Measures generation engine speed under parallel loads.
* <span class="chip">Queue Backlog</span> **Latency** — Helps decide when to spin up new GPU worker nodes.

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
* Active production monitoring dashboard configurations evaluate check karein.
* Tokens/sec performance speed metrics base thresholds levels verify check karein.
* GPU queue backlogs aur VRAM memory checks target scale configurations threshold monitor verify.
-->

---

# Common Problems and How to Fix Them

<div class="callout">
  <strong>⚡ Queue Congestion Latency</strong>
  Multiple incoming requests block each other, causing massive TTFB delays.<br>
  <span class="fix">→ Fix: Configure high concurrency environment setting <code>OLLAMA_NUM_PARALLEL</code>.</span>
</div>

<div class="callout">
  <strong>📦 Weight Transfer Network Bottlenecks</strong>
  Downloading models over external networks during container startup creates startup delays.<br>
  <span class="fix">→ Fix: Pre-bake model weight volumes into container layers or private storage mounts.</span>
</div>

<div class="callout">
  <strong>📉 Out of Memory (OOM) GPU Crashes</strong>
  Parallel requests exceed total hardware limits, triggering GPU process terminations.<br>
  <span class="fix">→ Fix: Implement request queuing buffers and configure strict VRAM fraction allocations.</span>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
* Final production gotchas and optimizations safety rules check karte hain.
* Pehla gotcha queue congestion ka hai—multitenant setups concurrency bottlenecks lock limits hit karte hain, isse override karne ke liye OLLAMA_NUM_PARALLEL environmental parameters standard check setup badhayein.
* Doosra issue weights loading delays network overhead ka hai—cold start delays avoid karne ke liye weights data volumes local disks layer mounts me store rakhein.
* Teesra out of memory GPU failures crash checks hai—memory allocations balance and protect configurations parameters configure karein.
* Is module ke sath humne course ke saare dimensions cover kar liye hain. Happy building!
-->
