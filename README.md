# 🧩 The Domino Effect Challenge

*A legendary RubixKube™ internship puzzle.*

Modern systems are dominos. One quiet failure can ripple across services and break things far away from the blast point. Your mission: **simulate this world**, **detect what matters**, **reason to root cause**, and **explain your thinking**.

This is not RubixKube. It’s about showing how you think, code, and reason about reliability. & Also It’s a proxy for **how you think under ambiguity**.

---

## 🎯 What you will build

A program (CLI or tiny web service) that:

1. **Simulates** a service graph over time (with random glitches)
2. **Detects** anomalies against a threshold
3. **Traces** downstream blast radius accurately
4. **Prioritizes** remediation (root cause first)
5. **Explains** the why with clear, human-readable output
6. **Records** an incident log we can review

Run locally. No cloud costs.

---

## 🧩 World model (formal spec)

* **Services graph**: Directed acyclic graph (DAG) by default, but your code **must** handle accidental cycles gracefully (detect & report).
* **Input file**: JSON array of services. Each service has:

  * `name: string` (unique, case-sensitive)
  * `depends_on: string[]` (zero or more upstream services)
  * `health: number` (initial value in [0.0, 1.0])

Example (`sample/services.json` provided):

```json
[
  { "name": "service-A", "depends_on": ["service-B", "service-C"], "health": 0.98 },
  { "name": "service-B", "depends_on": ["service-D"], "health": 0.95 },
  { "name": "service-C", "depends_on": ["service-E", "service-F"], "health": 0.99 },
  { "name": "service-D", "depends_on": [], "health": 0.97 },
  { "name": "service-E", "depends_on": ["service-G"], "health": 0.96 },
  { "name": "service-F", "depends_on": ["service-G", "service-H"], "health": 0.94 },
  { "name": "service-G", "depends_on": [], "health": 0.99 },
  { "name": "service-H", "depends_on": ["service-I"], "health": 0.92 },
  { "name": "service-I", "depends_on": [], "health": 0.97 },
  { "name": "service-J", "depends_on": ["service-B", "service-I"], "health": 0.95 }
]
```

---

## ⏱️ Simulation rules

* **Ticks**: The system evolves over `N` discrete ticks (iterations).
* **Glitches**: At most **one** random service may “glitch” per tick (unless you implement multi-glitch as a stretch). Glitch drops health by a random ∆ in `[0.2, 0.5]`.
* **Propagation** (downstream degradation): If an upstream dependency `U` falls below `threshold` (default `0.70`), each direct dependent `D` degrades by:

  * `D.health = max(0, D.health - α * (threshold - U.health))`
  * where `α` ∈ `[0.5, 1.0]` (choose and document your α; keep it constant for a run).
* **Recovery (optional)**: Auto-heal a failed service after a cooldown `K` ticks with `heal_to` (e.g., `0.85`). Recovery **must** propagate upstream improvements downstream (reverse blast).
* **Determinism**: Support `--seed <int>` so the same inputs produce the same outputs.

You can externalize parameters in `config.yaml`:

```yaml
ticks: 50
threshold: 0.70
alpha: 0.8
cooldown: 1
heal_to: 0.88
seed: 1337
```

---

## 🔎 Detection & RCA expectations

* **Anomaly**: A service is **FAILED** when `health < threshold`.
* **Blast radius**: All **downstream** nodes reachable from a failed node (via reverse-topology).
* **Root cause(s)**:

  * Prefer nodes with **no failed upstream** (i.e., failure originates there).
  * For multiple roots, order by **blast size** (more impacted first), then by **topological depth** (closer to the graph roots first).
* **Cycles**: If cycles are present, don’t crash. Detect and print a warning:

  * `[WARN] Cycle detected: service-X -> service-Y -> service-X (RCA may be approximate)`

---

## 🖥️ CLI contract (suggested)

Your tool should support:

```
run:    ./domino --input sample/services.json --config config.yaml
query:  ./domino "why is service-A failing?"
help:   ./domino --help
```

**Query semantics** (examples):

* `why is service-A failing?` → explain which upstream failed, when, and the chain.
* `what happened in the last 10 ticks?` → summarize incidents.
* `top-impacted` → list services by cumulative degradation.

If you build a web UI instead, document the endpoints (OpenAPI optional).

---

## 🧾 Output format (canonical)

Write a human-readable incident log to `./sample/output.log` (example already provided), and print key events to stdout. Suggested lines:

```
[ALERT] service-G fell below threshold (0.62 < 0.70) at T=2
[BLAST] due to service-G → impacted: [service-E, service-F, service-C]
[PRIORITY] roots={service-G} order=[service-G]
[SUGGESTION] Remediate service-G first
[HEAL] service-G -> 0.88 at T=3; recovered: [service-E, service-F, service-C]
```

Machine-readable **optional**: also emit `events.jsonl` with structured entries.

---

## ✅ Quality gates (we will look for)

* **Correct graph handling** (including out-of-order nodes, isolated nodes, cycles)
* **Determinism** with `--seed`
* **Clear RCA** (not just “who failed,” but **why** and **who was hit**)
* **Readable code** (modular, small functions, tests welcome)
* **Great README** (how to run, assumptions, trade-offs)

---

## 🧪 Scoring rubric (100 pts)

* **Reasoning & RCA (35)**: root-cause logic, blast accuracy, edge cases
* **Code quality (25)**: structure, naming, tests, docs
* **Practicality (20)**: easy setup, deterministic, handles real-ish data
* **Clarity of output (10)**: logs explain the story; a non-engineer can follow
* **Creativity (10)**: visualization, CLI queries, auto-heal, metrics, etc.

We don’t punish incomplete—but we **reward thoughtful**.

---

## 🚫 What not to do

* No external paid services or cloud infra
* No massive frameworks for a tiny CLI
* Don’t hide logic in black boxes—**show your thinking**
* Don’t copy/paste someone else’s solution (we can tell)

---

## 🕰️ Timing (read carefully)

**This challenge is open all the time — no fixed deadline.**
Use the **7-day window** as your personal measure of fairness: whenever you start, try to finish within 7 days and be honest about it in your submission. If you crack it now, great. If not, learn from it and come back later. **Our doors are always open for builders who love hard problems.**

---

## 📤 How to submit (no email needed)

1. Go to **Issues → New issue**
2. Choose **“Domino Effect Submission”** **[link](https://github.com/rubixkube-io/domino-effect-challenge/issues/new?template=submission.yml)**
3. Fill the form (public repo URL required) and submit

A GitHub Action will clone your repo, run basic checks, and comment results.

**Prefer a private submission?** Send details to **[connect@rubixkube.ai](mailto:connect@rubixkube.ai)**.

---

## 🧱 Repo template you can copy

* Keep your code under `/src` (or language-standard layout)
* Put configs in `/config` (optional)
* Put generated logs in `/runs/<timestamp>/`

We’ve included:

* `sample/services.json` (input)
* `sample/output.log` (example output)
* `.github/ISSUE_TEMPLATE/submission.yml` (issue form)
* `.github/workflows/validate_submission.yml` (basic checker)

---

## 🧠 Hints (not required, just helpful)

* Topo-sort for propagation; fall back to **Kahn’s algorithm** with cycle detection
* Precompute **reverse adjacency** for blast tracing
* Keep an `event_bus` abstraction so you can log & query easily
* Treat health as **float with clamping** `[0,1]`; avoid negative spirals
* For multi-root RCA, think **minimum cut intuition** (but keep it simple)

---

## 🏁 Why this matters

If your solution shows strong **reasoning, clarity, and care**, that’s RubixKube DNA.
If AI builds products, **you** help keep them alive.

Build it like you mean it. 🚀

---


**Good luck.** May the dominos fall in your favor.

