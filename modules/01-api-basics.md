---
marp: true
theme: default
paginate: true
header: 'Ollama in Your Project | Module 01'
footer: 'Software & AI Tools Course Series'
style: |
  section {
    background-color: #121214;
    color: #e1e1e6;
    font-family: 'Helvetica Neue', Arial, sans-serif;
  }
  h1, h2, h3 { color: #00b37e; }
  footer, header { color: #7c7c8a; font-size: 14px; }
  code { background-color: #202024; color: #e1e1e6; border-radius: 4px; }
  pre { background-color: #202024; border: 1px solid #29292e; }
  .columns { display: grid; grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 1rem; }
---

# <!-- fit --> Ollama API Architecture
### Master Module 01: `/api/generate` vs `/api/chat`

<!-- 
SPEAKER NOTES (Hinglish):
Hello engineers! Agar aap local LLMs ko apne full-stack core backend controllers me integrate kar rahe ho, toh standard desktop clients ya WebUI ka use karna band karo. 
Ollama background me ek persistent daemon architecture ke upar run hota hai, aur use code se target karne ke liye hamare paas do foundational REST endpoints hain: /api/generate aur /api/chat. 
Aaj hum in dono pipelines ke core differences aur execution mechanics ko line-by-line breakdown karenge.
-->

---

# The Core Architectural Division

<div class="columns">
<div>

### /api/generate
* **Execution Pattern:** Single-turn atomic completion.
* **Input Layer:** Flat prompt string.
* **State Management:** Fully stateless transaction.
* **Best Used For:** Automated parsing, strict text classification, code snippet writing.
</div>

<div>

### /api/chat
* **Execution Pattern:** Stateful multi-turn loops.
* **Input Layer:** Structured structural message arrays.
* **State Management:** Context managed via appended history.
* **Best Used For:** Interactive conversational pipelines, stateful agents.
</div>
</div>

<!-- 
SPEAKER NOTES (Hinglish):
Ab screen par dono pipelines ka architectural structural division dekho. 
Left side me hai generate endpoint—ye bilkul simple stateless worker hai. Ek clear task bheja, isne direct completion kiya, kaam khtam. Ko memory store nahi hoti. 
Right side me hai chat endpoint—ye stateful logic design ke liye bana hai, jahan structural conversational hierarchy maintenance zaroori hai. Iske bina multi-turn conversations handle nahi kiye ja sakte.
-->

---

# Payload Blueprint: `/api/generate`

```json
{
  "model": "qwen2.5-coder:7b",
  "prompt": "Write a Python function to check if a number is prime.",
  "stream": false,
  "options": {
    "temperature": 0.2
  }
}