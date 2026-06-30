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

# <!-- fit --> Local LLMs in Production

<div class="title-divider"></div>

### Scalability, Optimization, and Future Steps

<!-- 
SPEAKER NOTES (Hinglish):
Hey guys! Welcome to the final outro module of this course.
Humne is course me dynamic local LLM workflows, context window configurations, aur tool execution loops build karna seekh liya hai.
Ab jab aapka application local sandbox se nikal kar production level par move karega, toh scale-out architectures, concurrent scheduling, aur load balancing parameters ko handle karna hoga.
Chalo dekhte hain local configurations se production infrastructure me transitioning kaise execute hoti hai.
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
Chalo is scalability transition ke primary difference ko evaluate karte hain.
Left column me check karein Local Sandbox—ye hamara local setup hai jahan resources limited hain (jaise 4GB VRAM) aur runtime updates user-specific execution par restricted hain.
Right column me check karein Production Cluster—yahan dynamic scaling pipelines apply hoti hain. Model weights persistent nodes par load-balanced state me memory caches me store rehte hain, aur requests container orchestrator ke through distribute hoti hain.
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
Serving scaling architecture par focus karte hain.
Left side details me scale parameters check karein—load balancer configuration client connection distribute karti hai. Model load timeouts save karne ke liye keep_alive parameter always keep loaded setting `minus one` set kiya jata hai.
Right side architecture topology diagram me dekhein—dynamic load balancer requests distribute karke separate nodes (jaise multi-gpu server setups) par send karta hai, jisse compute overhead parallel state target criteria verify karta hai.
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
Production telemetry monitoring details check karte hain.
Left column metrics verify karein—scaling node validation check karne ke liye token generation speed throughput aur active request backlog metrics calculate kiye jate hain.
Right column check metrics trace—VRAM allocation limits and temperature parameters check parameters determine karte hain ki server hardware peak loads comfortably absorb kar raha hai ya system scale out threshold target compute limit check cross ho chuki hai.
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
Final production gotchas and optimizations safety rules check karte hain.
Pehla gotcha queue congestion ka hai—multitenant setups concurrency bottlenecks lock limits hit karte hain, isse override karne ke liye OLLAMA_NUM_PARALLEL environmental parameters standard check setup badhayein.
Doosra issue weights loading delays network overhead ka hai—cold start delays avoid karne ke liye weights data volumes local disks layer mounts me store rakhein.
Teesra out of memory GPU failures crash checks hai—memory allocations balance and protect configurations parameters configure karein.
-->
