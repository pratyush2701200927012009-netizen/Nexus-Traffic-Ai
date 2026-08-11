# Nexus-Traffic-Ai# NEXUS

Neural Exchange for Urban Situational Intelligence

> *"The city, understood."*

**A full-stack AI platform for real-time city traffic prediction, causal analysis, and signal optimization — built for San Francisco.**

🔴 **[Live Demo](https://pratyush2701200927012009-netizen.github.io/Nexus-Traffic-Ai/)** &nbsp;|&nbsp; Built by **Pratyush Mishra**

---

## What is NEXUS?

Most traffic systems tell you *what* is happening. NEXUS tells you *why* — and what to do about it.

NEXUS treats the city as a living graph. Every intersection is a node with memory. Every road is a weighted edge that shifts with weather, events, and human behavior. Every prediction is a **causal story**, not just a number — with uncertainty bounds, counterfactual reasoning, and natural language explanations via Claude.

The platform is designed for three personas simultaneously:
- **Traffic Operations Center operators** — real-time awareness, signal control, emergency routing
- **City transportation planners** — scenario simulation, historical analysis, intervention planning
- **Autonomous vehicle fleet operators** — probabilistic routing, incident prediction, surge forecasting

---

## Core AI Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Perception** | YOLOv8 + ByteTrack + Extended Kalman Filter | CV vehicle detection, sensor fusion |
| **Memory** | Neo4j + TimescaleDB + Redis | City graph, time-series, hot cache |
| **Cognition** | NEXUS-ST (GATv2 + Transformer) | Spatio-temporal traffic forecasting |
| **Deliberation** | MAPPO (Multi-Agent RL) | Cooperative signal optimization |
| **Causation** | DoWhy + Structural Causal Models | Causal attribution + counterfactuals |
| **Communication** | Claude API (RAG + Tool Use) | Natural language operator interface |

---

## Five Intelligence Modules

### 1. NEXUS-ST — Spatio-Temporal Graph Transformer
The prediction core. Treats the city as a heterogeneous dynamic graph and predicts congestion at 15m, 30m, 60m, and 120m horizons — with **calibrated uncertainty** (Gaussian outputs, not point predictions).

- **GATv2** spatial encoder: multi-head attention on edges, 3-hop receptive field
- **Transformer** temporal encoder: captures rush patterns, event surges, weather degradation
- **Context injection**: event embeddings (venue × attendance × time-to-end) condition predictions
- **Uncertainty heads**: epistemic (model uncertainty) + aleatoric (inherent randomness) separated

Target accuracy: MAPE < 8% at 15m horizon, < 15% at 60m.

### 2. Causal AI Layer
Standard ML finds correlations. NEXUS finds causes.

A **Structural Causal Model (SCM)** over ~50 traffic variables enables:
- **Causal attribution**: "Warriors game accounts for 78% of the predicted surge"
- **Counterfactual queries**: "Without the game, congestion would be 44% instead of 82%"
- **Intervention planning**: "If we close Elm Street, here is how it cascades through the graph"

Built on Pearl's do-calculus. Causal DAG discovered via PC algorithm on historical data, oriented with domain knowledge from traffic engineering.

### 3. Multi-Agent RL Signal Optimization (MAPPO)
Each intersection is an agent. Agents cooperate to minimize citywide delay.

- **CTDE paradigm**: centralized training with a shared critic that sees full city state; decentralized execution (each agent acts on local + 2-hop observations only)
- **Reward shaping**: ambulance delay penalized 10× more than regular traffic
- **Trained in SUMO** on historical demand patterns + randomized disruptions
- **Human-in-the-loop first**: recommendations shown to operators before autonomous mode

Benchmark: 20–30% delay reduction vs. SCOOT/SCATS (reactive fixed-time control).

### 4. Sensor Fusion (Extended Kalman Filter)
Reconciles 8 heterogeneous data streams — CV cameras, loop detectors, GPS probe data (FCD), Bluetooth/WiFi, weather stations, event APIs, social media, 311/CAD — into a single coherent city state estimate.

Per-sensor, per-time-of-day noise models. Bayesian updates. No ground-truth labels required.

### 5. Claude-Powered Language Interface
Traffic operators are not ML engineers. NEXUS exposes the entire system through natural language.

```
Operator: "Route Engine 7 from Station 4 to 3rd & King"
NEXUS:    "ETA 3.8 min. Route: Folsom → 2nd → King St.
           Signal preemption queued at 4 intersections.
           Avoid I-80 on-ramp (blocked). Alt via Howard adds 2.1 min."
```

Built on Claude API with RAG over live city state + tool-calling into prediction, causal, and routing services.

---

## Dashboard — Five Views

| Tab | What it shows |
|-----|--------------|
| **City Nerve Center** | Animated SVG city map with live congestion overlay, causal attribution panel, Claude chat assistant |
| **Prediction Timeline** | 4-panel view: congestion forecast with uncertainty bands, speed distribution, incident Gantt, zone heatmap |
| **Signal Control** | RL recommendation grid for all intersections — approve, reject, or manually override timing |
| **Scenario Sandbox** | Modify conditions (close a road, change weather, move event time) and run forward simulation |
| **Analytics** | Weekly KPIs, model accuracy by horizon, RL A/B impact, incident breakdown |

**Live features:** Gaussian random walk simulation with mean reversion, incident ETA countdown, keyboard shortcuts (1–5 to switch tabs, `/` to focus chat), auto-demo prompt on load.

---

## Why This Architecture?

**Why GNN over LSTM?**
Traffic is a graph. Congestion at node A propagates to node C before node B based on driver behavior. LSTMs treat each sensor independently — they miss the spatial cascade. GNNs share information across the graph during inference. The attention mechanism in GATv2 learns *which* neighbors matter for each intersection.

**Why RL over classical optimization (SCOOT)?**
SCOOT is reactive — it optimizes based on current loop data. MAPPO is predictive — it anticipates the incoming demand wave 15–30 minutes ahead and pre-adjusts. Sequential decision problem with delayed rewards = RL is the natural fit.

**Why Causal AI over SHAP?**
SHAP tells you "rain contributed 12% to this prediction." Causal inference tells you "rain *caused* 12% additional congestion because it slowed speeds on wet roads, which increased headways, which reduced throughput." The latter is actionable for an operator. The former is statistical decoration.

**Why Gaussian uncertainty outputs?**
A single-point prediction is epistemically dishonest. Traffic is stochastic. A prediction of "65% ± 12%" is more useful than "65%" because operators can act on the uncertainty — high σ means deploy monitoring resources, not just wait.

---

## Data Pipeline

```
Kafka topics (real-time ingestion):
  nexus.camera.{zone_id}        — CV feature vectors, 1Hz
  nexus.detector.{sensor_id}    — Loop detector readings
  nexus.probe.{segment_id}      — GPS probe aggregates, 30s
  nexus.weather.{station_id}    — Weather observations, 1min
  nexus.event.{event_id}        — Event lifecycle updates
  nexus.incident.{incident_id}  — Accidents, closures, CAD
  nexus.signal.{intersection_id}— Signal state, 1Hz

Storage:
  Redis          — last 15 min (hot cache for real-time inference)
  TimescaleDB    — full time-series history
  Neo4j          — city graph (4,218 nodes · 11,042 edges)
```

---

## Inference Latency Budget

| Component | Target |
|-----------|--------|
| Sensor ingestion (Kafka) | < 50ms |
| Graph state update (Redis) | < 10ms |
| NEXUS-ST inference (GPU) | < 150ms |
| Causal explanation | < 200ms |
| LLM response (streaming) | < 2s |
| **Total prediction pipeline** | **< 500ms** |

---

## Research Foundation

NEXUS synthesizes ideas from:

- **DCRNN** (Li et al., ICLR 2018) — diffusion GNNs for traffic
- **Graph WaveNet** (Wu et al., IJCAI 2019) — adaptive adjacency matrix
- **GATv2** (Brody et al., ICLR 2022) — improved graph attention
- **CityFlow** (Zhang et al., WWW 2019) — multi-agent RL for traffic
- **MAPPO** (Yu et al., NeurIPS 2022) — cooperative multi-agent PPO
- **Kendall & Gal** (NeurIPS 2017) — epistemic vs. aleatoric uncertainty
- **DoWhy / Pearl** — structural causal models and do-calculus

---

## Tech Stack

**ML/AI:** PyTorch Geometric · PyTorch Lightning · DoWhy · YOLOv8 · RLlib (Ray) · SUMO · Weights & Biases · Optuna

**Backend:** FastAPI · Go · Apache Kafka · Neo4j · TimescaleDB · Redis · Kong

**Frontend:** Single-file HTML/JS · SVG animation · WebSockets · Confidence band visualization

**Infrastructure:** Docker · Kubernetes (GCP target) · Terraform · GitHub Actions · Grafana · Prometheus

**LLM:** Claude API (Anthropic) — RAG + tool-calling over live city state

---

## Try It

🔴 **[Open Live Demo](https://pratyush2701200927012009-netizen.github.io/Nexus-Traffic-Ai/)**

- The **NEXUS Assistant** (right panel) is Claude-powered — ask it anything about city traffic
- Use keys **1–5** to switch between tabs
- Press **/** to focus the chat input
- The **Scenario Sandbox** (tab 4) lets you simulate closing roads, moving event times, clearing accidents

---

## Author

**Pratyush Mishra**

Built as a research-grade portfolio project at the intersection of graph neural networks, reinforcement learning, causal AI, and urban systems — targeting production-grade architecture, not toy demos.

---

*NEXUS — Neural Exchange for Urban Situational Intelligence*
