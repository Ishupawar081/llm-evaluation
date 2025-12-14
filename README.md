# LLM Response Evaluation 

##  Overview

This project implements a **simple, lightweight framework to evaluate Large Language Model (LLM) responses** using only core Python and basic NLP techniques.

The goal is to analyze chatbot answers and measure:
- How relevant the response is to the user’s question
- Whether the answer is complete
- Whether the response hallucinates information not present in context
- Approximate token usage and cost

This project is **model-agnostic** and does not require access to any LLM API.

---

## Local Setup

### Requirements
- Python 3.9+
- No external libraries required

### Steps
1. Clone the repository:
   ```bash
   git clone <repo-url>
   cd llm-evaluation

---

###  Architecture of the Evaluation Pipeline

```markdown
 Evaluation Pipeline Architecture

The evaluation system follows a modular, linear pipeline:

1. Input Layer  
   - Chat conversation JSON  
   - Retrieved context / knowledge JSON  

2. Parsing Layer  
   - Attempts strict JSON parsing  
   - Falls back to regex-based extraction if JSON is malformed  

3. Turn Extraction  
   - Extracts last user query  
   - Extracts corresponding AI response  

4. Metric Engine  
   - Relevance (cosine similarity)
   - Completeness (keyword coverage)
   - Hallucination risk (entity & number grounding)
   - Token usage & cost estimation  

5. Output Layer  
   - Aggregated numerical evaluation scores
```

## Design Rationale

This solution was intentionally built using lightweight, rule-based methods instead of
LLM-based or embedding-based evaluators.

Reasons:
- Real-world chat logs are often malformed → regex fallback ensures robustness
- Embeddings and LLM judges introduce latency and cost
- Rule-based metrics are deterministic and explainable
- The system remains model-agnostic and easy to debug
- No external APIs → no rate limits or vendor lock-in

## Scalability Considerations

This framework is designed to handle millions of daily conversations:

- No LLM inference calls → zero per-request model cost
- No embeddings or vector databases
- All metrics run in linear time O(n)
- Stateless design enables easy horizontal scaling
- Can be parallelized across CPU workers
- Suitable for real-time or batch evaluation

As a result, latency per conversation remains in milliseconds and operational cost stays minimal.



## Why this project?

Evaluating LLM outputs is difficult because:
- Fluent answers may still be incorrect
- Chat logs often contain invalid or inconsistent JSON
- Ground-truth answers are not always available

This framework focuses on **practical evaluation**, similar to real production systems, and works even when data is noisy or partially broken.

---




##  Metrics Implemented

###  Relevance to User
Uses **cosine similarity** between tokenized user query and AI response.

- High score → AI stayed on-topic
- Low score → Response drifted

---

### Completeness
Checks how many important keywords from the user query appear in the AI answer.

- High score → User intent covered well
- Low score → Missing important details

---

###  Hallucination Risk
Estimates hallucination by comparing:
- Named entities
- Numbers

between the AI response and retrieved context.

- High score → High hallucination risk
- Low score → Response grounded in context

---

###  Token Usage & Cost
Approximates token count using word-based tokenization and estimates cost using a fixed per-token rate.

This helps analyze response efficiency.

---

## Handling Invalid JSON (Important Feature)

Many real-world chat logs are **not valid JSON** due to:
- Comments (`// ...`)
- Trailing commas
- Missing fields

Instead of failing, this framework:
1. Tries strict JSON parsing
2. Falls back to **regex-based extraction**
3. Switches to **text-only evaluation mode**

This makes the system **robust and production-realistic**.

---

##  How to Run (Google Colab or Local)

### Step 1: Upload / Clone Repository
Upload files to Colab or clone locally.

### Step 2: Run Evaluation
```bash
python run_evaluation.py
```

### Sample Output
```bash
JSON parsing failed, using regex fallback

Evaluation Results:

relevance_to_user: 0.1971
completeness: 0.6667
hallucination_risk: 0.32
tokens: 83
estimated_cost_usd: 0.0002
```

