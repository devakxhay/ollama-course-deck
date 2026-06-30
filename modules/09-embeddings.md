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

<span class="module-label">Module 09 · Embeddings</span>

# <!-- fit --> <img src="../assets/ollama.png" height="60" style="vertical-align: middle; margin-right: 15px;" /> Embeddings & Semantic Search

<div class="title-divider"></div>

### Vectorizing Text for Similarity and Retrieval

<!-- 
SPEAKER NOTES (Hinglish):
* Welcome back! Module 08 me humne structural FIM structures aur code autocompletion templates override methods check kiye.
* Par local knowledge bases, documentation datasets aur matching applications text vectors calculations base scale demand karti hain.
* Aaj hum text documents ko semantic float arrays me transform karne ke liye Embeddings APIs lookup logic evaluate karenge.
* Hum literal keyword lookup aur geometric vector similarity patterns ke differences check karenge.
-->

---

# Keyword vs. Semantic Search: What is the Difference?

<div class="columns">
<div class="col">

### Keyword Search
* **Matching Criteria:** Finds exact string character matches.
* **Limitations:** Misses synonyms or concepts (e.g. "car" vs "automobile").
* **Algorithms:** SQL LIKE query, Elasticsearch BM25.

</div>
<div class="col">

### Semantic Search
* **Matching Criteria:** Measures geometric distance between text vectors.
* **Benefits:** Matches meaning and context irrespective of vocabulary.
* **Algorithms:** Cosine Similarity, Vector Database indexing.

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
* Keyword search aur Semantic vector search processes compare analyze karein.
* Left side keyword checks exact string parameters syntax mapping par verify criteria run karta hai.
* Right side semantic metrics dynamic conceptual coordinates map vector databases logic update select karta hai.
-->

---

# How to Generate Embeddings in Go

```go
// 1. Configure the payload calling the embeddings service
payload := map[string]interface{}{
    "model":  "nomic-embed-text",
    "prompt": "Local LLMs in production",
}
// ... Send HTTP POST request to /api/embeddings ...

// 2. Decode the floating point vector array output
var result struct {
    Embedding []float64 `json:"embedding"` // High-dimensional float array
}
json.NewDecoder(resp.Body).Decode(&result)
```

<!-- 
SPEAKER NOTES (Hinglish):
* Go program models endpoints embedding logic implementation configurations check karein.
* Standard generation workflows skip `/api/embeddings` target parameters request structure apply karte hain.
* Response return coordinates float slices vector database setup standard indexing me pass hotey hain.
-->

---

# How to Read Embedding Telemetry

<div class="columns">
<div class="col">

### Telemetry Payload
```json
{
  "embedding": [
    0.012543,
    -0.08235,
    "..."
  ]
}
```

</div>
<div class="col">

### Telemetry Check
* <span class="chip">embedding</span> **Float Array** — A vector of numbers representing the semantic concepts of the text.
* <span class="chip">dimensions</span> **Vector Size** — Nomic model generates 768 dimensions per vector.

</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
* Telemetry response embeddings configurations metrics details parse analyze karein.
* Left side output parameters direct array values coordinate dimensions hold cards populate karta hai.
* High-dimensional structures similarities checks and geometric distances calculations parameters verify logic set.
-->

---

# Common Problems and How to Fix Them

<div class="callout">
  <strong>🚫 Model Dimension Mismatches</strong>
  Comparing vectors generated by different models throws runtime errors.<br>
  <span class="fix">→ Fix: Enforce a single embedding model (e.g. <code>nomic-embed-text</code>) for the entire index database.</span>
</div>

<div class="callout">
  <strong>📏 Document Size Context Loss</strong>
  Embedding massive files directly compresses all meaning into a single generic vector.<br>
  <span class="fix">→ Fix: Chunk long documents into smaller segments (200-500 tokens) using overlapping windows.</span>
</div>

<div class="callout">
  <strong>⚡ Slow CPU Search Bottlenecks</strong>
  Comparing query vectors against millions of items using simple loops locks server CPUs.<br>
  <span class="fix">→ Fix: Deploy specialized Vector Databases (e.g., <code>pgvector</code> or <code>Qdrant</code>).</span>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
* Vector databases integrations issues and optimization setups checks apply karein.
* Dimension mismatches avoid parameters single constant model configuration rule maintain set verify.
* Chunking logic implement details context loss prevent bounds options parameters setup apply karein.
-->

---

# Next Up: Capstone (Building a Local Agent)

<div class="title-divider"></div>

### Next Up: [Module 10 · Capstone Project](./10-capstone.html)

* **What we will cover next:**
  * Orchestrating the Reasoning & Acting (ReAct) loop on local hardware
  * Combining semantic search (RAG) and tool calling inside a single agent cycle
  * Tracking token scaling and latency telemetry during reasoning loops
  * Optimizing agents for lower-spec GPUs like the GTX 1650 (4GB VRAM)
* **Get ready to bring all modules together into a working local agent!**

<!-- 
SPEAKER NOTES (Hinglish):
* Embeddings vector configurations aur semantic search setup successfully completed.
* Ab tak humne isolated capabilities build kiye hain par dynamic AI agents autonomous behaviors loops require karte hain.
* Agle module (Module 10) me hum custom cognitive loops build karne ke liye full capstone project autonomous agent framework design karenge. Let's start!
-->

