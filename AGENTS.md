# Course Framework & Rulebook: "Ollama in Your Project"

## 1. Technical Persona & Presentation Strategy
*   **Target Audience:** Software developers, DevOps engineers, and system architects.
*   **Tone:** Production-oriented, high density, authoritative, zero conversational fluff.
*   **Content Philosophy:** Focus strictly on backend logic, raw performance metrics, and configuration code over surface-level UI configurations.

## 2. Language Architecture (The Hinglish Strategy)
*   **Visual Slide Content:** 100% Technical English. Use scannable headers, precise code terminology, and minimalist bullet layouts.
*   **Speaker Notes (Hidden via Comments):** Pure Hinglish following a "Code in English, Explanations in Hinglish" delivery style. Keep descriptions conversational but accurate. Use standard engineering loan words (*payload, buffer, request, overhead*).

## 3. Visual & Layout Guidelines (Marp Standards)
*   Each presentation file must begin with standard global directives: `marp: true`, `paginate: true`, and clean presentation themes.
*   **Slide Dividers:** Separate slide boundaries strictly using a single three-dash sequence (`---`).
*   **Code Containers:** Wrap all syntax implementations inside Markdown blocks with explicit language highlighting tags (e.g., `bash`, `json`). Ensure code formatting stays clean without overflowing visual boundaries.
*   **Structure Sequence:**
    1. Title & Hook Slide
    2. Architecture Core Concept
    3. Payload Configuration / Code Snippet
    4. Execution Telemetry / Output Analysis
    5. Production Gotchas / Error Resolution