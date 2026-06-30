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

# <!-- fit --> Embeddings & Semantic Search

<div class="title-divider"></div>

### Vectorizing Text for Similarity and Retrieval

<!-- 
SPEAKER NOTES (Hinglish):
Hey guys! Welcome back. Aaj hum baat karenge local databases search ke ek bohot important topic ke baare me—wo hai Embeddings & Semantic Search.
Inference generate karne ke sath, hume documents search karne, similarities verify karne, ya RAG pipelines build karne ke liye text data ko vector formats me map karna padta hai.
Is module me hum seekhenge ki kaise Ollama embeddings endpoint utilize karte hain aur context metrics retrieve karte hain.
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
In dono search types ke internal difference ko evaluate karte hain.
Left column check karein Keyword Search—ye basic character matching par focused hai. Agar text matching exact nahi hui toh queries fail ho jati hain.
Right column check karein Semantic Search—yahan models words ko embedding values vector map coordinates me convert karte hain. Isse exact characters mismatch hone par bhi, meaning matching base similarity score check ho jata hai.
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
Code codebase side implementation check karte hain.
Ollama me generate or chat API ke bajaye specialized API endpoint `/api/embeddings` call hota hai.
Payload me hum prompt text aur specialized model name pass karte hain. HTTP response return hone par, body property `embedding` key check karein—ye ek long float slice return karta hai jo text data point coordinates target space maps representation hold karta hai.
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
Embedding outputs and metrics details parse karte hain.
Left side JSON check karein: response parameter me hume numeric values list receive hoti hai.
Right side check karein: array length coordinate dimensions determine karti hai. Jaise nomic-embed-text structure 768 float values map structure follow karta hai. In dimensions vectors ko hum cosine similarity algorithms run karke index similarity metrics calculate karte hain.
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
Vector search pipelines production gotchas check karein.
Pehla problem dimension scale mismatch hai—different models vector properties different size generate karte hain. Hamesha search index me constant model use karein.
Doosra bottleneck document length issue hai—bade page size files direct vectors convert karne par precision metrics destroy ho jati hain, isliye overlapping sliding chunk logic apply karein.
Teesra speed scaling constraints hai—millions records loops evaluation slow ho jati hain, iske liye indexing vector DB options execute karein.
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
Chalo ab next module par chalte hain! Agle class me hum seekhenge ki kaise in saare building blocks ko connect karke ek self-reasoning local agent build kiya jata hai aur use GTX 1650 specification limits ke according optimize karte hain. See you in the next module!
-->

