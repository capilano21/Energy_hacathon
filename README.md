# 🔄 Grid Forecast + Load Shedding Simulator

## 📌 Project Title & Description

This simulator models a decentralized smart grid system capable of proactive load shedding. It leverages transformer-level forecast data, evaluates potential overloads using LLM reasoning, and escalates to an RL control agent to recommend shedding strategies. Users are notified using suggestion, request, or command formats compliant with the Beckn protocol.

All data used for simulation—including transformer locations, DER mappings, and user-device relationships—is sourced from the **World Engine backend**, connected via **Strapi APIs**.

---

## 🌐 Live Demo

🎯 Try the interactive Gradio simulator here:  
https://huggingface.co/spaces/bluegrey/energy_agentic

---

## 👥 Team Members

* Ashwin Thyagarajan
* G Mohan
* Rajkumar V

---

## 🔀 Tech Stack Overview

Our system is built to implement an **agentic grid orchestration framework** for managing energy consumption, DER load shedding, and behavioral incentive alignment using:

* **OpenAI GPT-4.1** with structured output and tool calling
* **Time Series Forecasting models** (Prophet, TimeSFormer, MAML-based adaptive models)
* **Reinforcement Learning (RL)** agents with future support for DDPG/DDQ
* **Beckn Protocol** for dynamic interactions and reward-based catalogs
* **Gradio** for interactive interface
* **Pydantic** for schema validation
* **Pandas** for transformer and user data handling
* **Strapi APIs** from World Engine to access real-time transformer and DER metadata
* **Custom Simulation Backend** using Python

This hybrid architecture enables self-monitoring agents to personalize decisions, evolve policies, and optimize user engagement.

---

## 🧠 GPT-4.1 Reasoning Engine

### ⚖️ Structured Output + Tool Calling

* **Tool Calling**: GPT-4.1 invokes utilities like transformer-load lookups, user mapping, reward logs
* **Pydantic Schemas**: Used to enforce valid, structured return formats
* **Roles**: Forecast assessor, escalation manager, DER recommender, reflection summarizer

### 🧠 Memory and Self-Reflection

GPT-4.1 maintains semantic memory of past actions:

* Tracks DER success/failure
* Logs compliance & reward
* Detects overuse/misuse of certain tones (e.g., over-commanding)
* Refines future decisions using past window summaries

**Self-Prompted Summary**:

```markdown
## What Worked?
## What Should Be Avoided?
## Policy Refinement for Next Round
```

---

## ⛅ Forecasting Strategy

### 1. Prophet

* Fast, interpretable
* Cyclical load forecast

### 2. TimeSFormer (State-of-the-Art)

* Transformer-based model for time series
* Captures spatiotemporal correlations across grid

### 3. MAML

* Few-shot adaptation to new regions
* Effective in data-sparse setups

---

## 🔁 RL Escalation and Load Shedding

### 🚨 Triggering RL

* Individual transformer load > 1.10 kWh
* Global demand > 3.25 kWh

### 🧠 RL-Bot #1 — Transformer Selection

Selects transformers with high load + poor compliance history

```json
{"transformer_ids": ["TX-001", "TX-003"]}
```

### 📦 RL-Bot #2 — DER Messaging

For each transformer, maps DER to user and selects tone:

```markdown
| User | DER | Message |
|------|-----|---------|
| Lisa Kim | Microwave | suggestion |
| Brad Cook | Geyser | command |
```

### 🔮 Compliance Simulation

* Probabilistic
* Sensitive to prior rewards
* Models user fatigue and habits

### 🏅 Reward Assignment

```json
{"User": "Brad Cook", "Reward": 2}
```

Rewards decay for ineffective DERs or high messaging frequency.

---

## 📉 Beckn Protocol Integration

### 🎯 Electricity Market Simulation

#### /search

User requests connections. BPP returns base offers.

#### /on\_search

We modify catalogs to reflect dynamic **reward-based pricing**:

```markdown
| User        | Reward | Discount % | Final Price |
|-------------|--------|-------------|--------------|
| Lisa Kim    | 5      | 20%         | $64          |
| Adam Conley | -1     | 0%          | $80          |
```

---

## 🔄 Semantic Memory and Policy Evolution

### 📂 Memory Types

* **User-Level**: DERs, reward, tone history
* **Transformer-Level**: overload events, successful mitigations

### 🔐 Keyword Summarization

Users like "Stephanie Stephens" are deprioritized if overtargeted.

### ✨ Meta-Learning Integration

* **DDPG** for partial-compliance learning
* **Off-policy simulators** to scale interactions
* **LLM-refined gradients** to fuse human-friendly explanations with policy learning

---

## 🌱 Outcome Impact

* ✅ Maintains grid stability
* ✅ Rewards users transparently
* ✅ Evolves through memory + reflection
* ✅ Minimally intrusive interventions

---

## 📘 Setup Instructions

```bash
pip install gradio openai pandas pydantic
export OPENAI_API_KEY="sk-..."
python final_clean_gradio_simulator.py
```

Launches a local Gradio interface.

---

## 📖 Challenges & Learnings

* Designing multi-role GPT-4 prompts
* Reward shaping with Beckn pricing
* Building RL feedback loops inside LLM pipeline
* Avoiding user fatigue using adaptive tones

---

## 🔗 Useful Resources

* [Beckn Protocol](https://becknprotocol.io)
* [OpenAI Function Calling](https://platform.openai.com/docs/guides/function-calling)
* [Gradio](https://gradio.app)
* [Pydantic](https://docs.pydantic.dev/latest/)

---

## 📥 Beckn Protocol Reward Catalog Logic

```python
def beckn_discount_pipeline():
    import requests, json, copy

    # CONFIG
    bearer_token = BEARER_TOKEN
    bap_client_url = "http://bap-ps-client-deg-team6.becknprotocol.io"
    ...

    # BASE CATALOG REQUEST
    res = requests.post(...)
    catalog = res.json()["responses"][0]["message"]["catalog"]

    # USER REWARDS
    user_rewards = {r["User"]: r["Reward"] for r in latest_rewards_log}

    # APPLY DISCOUNTS
    def reward_to_discount(r): return 0.2 if r >= 5 else 0.1 if r >= 2 else 0.0
    ...

    for user, reward in user_rewards.items():
        personalized = apply_discounts(copy.deepcopy(catalog), reward)
        print(json.dumps(personalized, indent=2))
```

Catalogs are customized per-user before returning `on_search` results.

---

## 🚀 Future Directions

* Integrate real smart meter feeds
* Scale to 15+ transformer clusters
* Enable `/on_update` and `/status` lifecycle
* Live dashboard for RL actions and reward logs
* Build policy gradient agents trained with LLM traces

---

## 📏 System Flow Diagram

```text
[Forecasting (Prophet/TimeSFormer)]
           |
           v
[Overload Detection (Threshold Check)]
           |
           v
[LLM Evaluation (GPT-4.1 Tool Call)]
           |
           v
[RL Trigger] ---> [Transformer Selection]
           |
           v
[User-DER Mapping]
           |
           v
[Message Plan (Suggestion/Request/Command)]
           |
           v
[Simulate Compliance (Probabilistic)]
           |
           v
[Assign Rewards] ---> [Semantic Memory Update]
           |
           v
[Catalog Discount (Beckn /on_search Update)]
```

---

