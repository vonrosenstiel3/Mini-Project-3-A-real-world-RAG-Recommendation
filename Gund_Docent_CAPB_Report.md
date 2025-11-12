# Mini-Project #3: Real-World RAG Implementation  
**Course:** AI Mini-Project Series  
**Due Date:** Nov 12  
**Deliverable:** Completed CAPB Report

---

## 1. Project Context & Use Case

### 1.1 Problem / Context
Build a **RAG-backed docent agent** for **The Gund (Gund Gallery, Kenyon College)** that answers visitor questions about the permanent collection and current exhibition, explains art-theory links, and provides provenance/donor context when grounded in official materials. It adapts tone and reading level, supports multiple languages, and ensures accessibility and factual grounding.

### 1.2 Primary Use Case
- [x] Research assistant / document summarization  
- [x] Q&A over collection/exhibition info  
- [x] Art-theory connection and recommendation engine  

### 1.3 Users / Audience
- Kenyon students and faculty (academically curious or expert)  
- Local community visitors (general audience)  
- K–12 school groups (children)  
- Tourists and multilingual users  
- Visually-impaired and low-literacy users  

### 1.4 Success Criteria

```yaml
context:
  domain: "Museum collection & exhibitions (The Gund)"
  use_case: "AI docent for collection Q&A and connections"
  users: ["students", "faculty", "local visitors", "children", "tourists"]
  success:
    - "≥85% correct grounded answers"
    - "≥90% answers include citations"
    - "≤4s average latency on 20–50 page corpus"
    - "Adaptive tone and reading level"
    - "Multilingual support (≥3 languages)"
    - "Zero hallucinated provenance claims"
```

---

## 2. Data & Constraints

### 2.1 Corpus Details
- **Sources:** The Gund public website + exhibition PDFs (didactics, labels)  
- **Focus:** Permanent collection with supplementary current exhibition content  
- **Size:** 20–50 pages total  
- **Formats:** HTML → text and PDFs converted to text  

### 2.2 Constraints

```yaml
data_constraints:
  sources: ["Gund public website", "exhibition PDFs"]
  formats: ["html->txt", "pdf"]
  size: "20-50 pages"
  local_only: true
  cost_limit: "OSS/low-cost"
  latency_target: 4
  security: "no PII; app login via museum app"
```

---

## 3. RAG Architecture (MVP)

### 3.1 Pipeline
Ingest → Clean → Chunk → Embed → Vector DB → Hybrid Retrieval → Rerank → LLM → Citation-grounded answer → (Optional) LightRAG explanation

### 3.2 Core Design Choices

```yaml
architecture_mvp:
  steps: ["ingest", "clean", "chunk", "embed", "retrieve_hybrid", "rerank", "generate", "cite"]
  chunking: "heading+sentence window (~600 tokens)"
  embeddings: "bge-m3 (multilingual, local)"
  vector_db: "chroma"
  retrieval: "hybrid (BM25 + vector)"
  reranker: "bge-reranker-large"
  llm: "Llama 3.1 8B Instruct (local/Ollama)"
  citations: true
  rationale: "Grounded, multilingual, local pipeline with low hallucination risk"
```

### 3.3 Rationale for LightRAG
The **LightRAG** architecture is chosen to explain relationships between artists, artworks, and themes, without the overhead of full GraphRAG. It can model connections such as *Artist → Theme → Movement → Donor*, improving interpretability and recommendation quality.

---

## 4. Component Alternatives (Mini-Bakeoff)

| Component | Option A | Option B | Criteria | Selected | Why |
|------------|-----------|-----------|-----------|----------|-----|
| Embeddings | bge-m3 | gte-large | multilingual, OSS | **bge-m3** | multilingual and accurate |
| Vector DB | Chroma | FAISS | local setup, dev ease | **Chroma** | simple persistence |
| Retrieval | Hybrid | Vector-only | precision vs simplicity | **Hybrid** | supports art text nuance |
| Reranker | bge-reranker-large | Cohere | cost vs performance | **bge-reranker** | free + robust |
| LLM | Llama 3.1 8B | GPT-4o-mini | local cost | **Llama local** | grounded and offline |
| Graph Layer | LightRAG | GraphRAG | complexity vs interpretability | **LightRAG** | lightweight, efficient |

---

## 5. Evaluation Plan & Results

### 5.1 Gold Test Set
15 representative questions testing correctness, citation, tone, and relationship explanations.

**Examples:**  
- *Child visitor:* “Why is this painting special? Tell me like I am 8.”  
- *General visitor:* “What themes connect these two works from the collection?”  
- *Museum professional:* “How does this work situate itself within post-minimalist practice? Discuss its material strategies.”  
- *Art student:* “Which artists in the collection share feminist theoretical concerns?”  
- *Sensitive topic:* “Was this work ever part of a colonial collection?”  
- *Accessibility:* “Describe this piece for someone who cannot see it.”  
- *Multilingual:* “Explícalo en español, por favor.”  

### 5.2 Metrics

```yaml
evaluation:
  questions: 15
  metrics:
    - "Correctness ≥85%"
    - "Citations ≥90%"
    - "No provenance hallucinations"
    - "Latency ≤4s local"
    - "Tone adaptation pass/fail"
  procedure:
    - "Manual review by 2 evaluators"
    - "Citations logged and verified"
```

---

## 6. Risks, Edge Cases & Future Work

### 6.1 Risks
- Hallucinations in provenance/donor data  
- Oversimplification of complex history  
- Reading-level mismatch  
- Multilingual misinterpretation of theory terms  

### 6.2 Mitigation
- Strict grounding and “not in sources” fallback  
- Sensitive-topic safe mode (verbatim only)  
- Hybrid retrieval with reranker  
- LightRAG for explainable relationships  
- Reading-level classifier and adaptive tone control  

### 6.3 Future Work

```yaml
improvements:
  - "Deploy LightRAG fully for artist-theme graphs"
  - "Add incremental reindexing of new exhibitions"
  - "Create multilingual art-term glossary"
  - "Add voice-interaction and VI-friendly mode"
  - "Integrate RAGAS automated evaluation"
```

---

## 7. Prompts

### System Prompt
```
You are the Gund’s AI docent. Use only provided museum sources. No speculation.  
For sensitive topics (colonialism, provenance, remains, donors), quote or closely paraphrase.  
If information is missing, say: “I don’t have that in my sources.”  
Always cite sources. Adapt reading level and language. Maintain a warm, docent-like tone.
```

### Persona Variants
**Child:** “Explain ideas simply, with analogies.”  
**Professional:** “Use precise art-historical and material terminology.”  

### Safety Prompt
“Do not infer donor intent or provenance. Never speculate.”

---

## 8. Implementation Outline

```yaml
framework: "LangChain or LlamaIndex"
ingest: ["HTML pages", "museum PDFs"]
store: "Chroma"
rerank: "bge-reranker-large"
llm: "Llama 3.1 8B (Ollama)"
graph: "LightRAG JSON graph (artists, themes, donors)"
output: "citations + snippet + fallback if unknown"
```

---

## 9. Personas

```yaml
personas:
  child: "grade 4–6; playful, clear language"
  general: "friendly, concise"
  student: "college-level contextual detail"
  professional: "rigorous art-theory tone"
  multilingual: "en/es/fr/zh"
  low_vision: "visual description based on label text"
```

---

## 10. References
- LightRAG documentation and benchmarks  
- “15 Best Open-Source RAG Frameworks in 2025” by Tuychiev  
- GraphRAG resources (Awesome lists)  
- LangChain / LlamaIndex docs  
- Sentence-transformers (bge-m3, reranker)  
- The Gund official site and exhibition PDFs  

---
