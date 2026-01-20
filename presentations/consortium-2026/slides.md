---
theme: default
title: MOAF-DiT — Formalising Process Knowledge as an Ontology
info: |
  Annual consortium meeting - 20 January 2026
  Presenter: Arghavan Akbarieh (TU/e)
class: text-center
---

# MOAF-DiT 
## Formalising Process Knowledge as an Ontology

<div style="margin-top:24px; opacity:0.85">
Annual consortium meeting - 20 January 2026
</div>

---

# Why this project?
- PCB manufacturing is **high-mix / high-variance** (board types, test pathways)
- Operational questions are currently answered via **documents, spreadsheets, and tacit knowledge**
- Goal: make process + routing + runtime execution **queryable**, **explainable**, and **linkable to a 3D digital twin**

---

# Target capability
Users (engineers, supervisors) should be able to ask:

- “Where is WIP board X right now?”
- “What is the next step and department?”
- “Who is responsible at this moment (role)?”
- “Which route does BoardType Y follow?”
- “Why is this board delayed (trace narrative)?”

And receive: **structured answers + natural-language explanations**.

---

# Inputs (evidence base)
1) Karel use-case documentation & scenarios
2) Factory videos / walkthrough understanding
3) Exploratory data analysis (EDA) report (orders, board types, routing patterns)
4) Shop-floor API / Excel extracts (execution-level data)
5) BPMN modelling of departmental processes

---

# Method overview (end-to-end)
1. Collect domain knowledge (docs, videos)
2. Analyse datasets (EDA) to discover **board families/types/routes**
3. Model processes in **BPMN** (departmental view)
4. Translate into ontology: **Process/ProcessStep/Role/Resource**
5. Extend with product routing: **Route/RouteStep**
6. Add runtime tracking: **ExecutionStep**
7. Iterate: constraints, inverses, naming, competency questions
8. Prepare for Digital Twin + GraphRAG integration

---

# Key modelling decision: BPMN rectangle = ProcessStep
- **ProcessStep** is the atomic activity (one BPMN task/rectangle)
- Each ProcessStep:
  - belongs to one department/process
  - requires roles and resources
- This preserves BPMN meaning and supports querying at “activity” level

---

# Three layers: Process vs Route vs Execution
We separated three different “jobs” that were previously mixed:

1) **Process** (departmental capability)
2) **Route** (product-specific path across departments)
3) **Execution** (runtime state for a specific WIP board)

This separation is the core enabler for accurate queries.

---

# Layer 1: Process (departmental view)
- Department **hasProcess** Process
- Process **hasProcessStep** ProcessStep
- ProcessStep **requiresRole** Role
- ProcessStep **requiresResources** Resource/Machine/Material

**Interpretation:** what *can* happen and how it is organised inside each department.

---

# Layer 2: Route (product-specific view)
A Route answers:
- which departments/processes a BoardType goes through
- in what order
- including repeats and branches (e.g., Testing ↔ Repair loops)

**Route** consists of ordered **RouteSteps**.

---

# RouteStep: why it exists
RouteStep represents a **position** in a route, not a BPMN rectangle.

Example: BoardType_X route:
Material → Assembly → Wave → AOI → Testing → Repair → Testing → Packaging

- RouteStep_X_5 = “Testing at position 5 for BoardType_X”
- RouteStep_X_6 = “Repair at position 6 for BoardType_X”

Each RouteStep **refersToProcessStep** (entry-level step or process anchor).

---

# Layer 3: ExecutionStep (runtime)
ExecutionStep represents:
> the runtime execution of a RouteStep for a specific WIP board

- ExecutionStep **forWIPBoard** WIPBoard
- ExecutionStep **executesRouteStep** RouteStep
- (optional) timestamps, status, machine used

**Interpretation:** what is happening *now* (or happened) for traceability.

---

# Example query chain (runtime location)
To answer “Where is WIP board 983472 now?”:

ExecutionStep (current)
→ executesRouteStep
→ refersToProcessStep
→ belongsToDepartment

This yields: **current department + current activity + responsible roles/resources**.

---

# Karel routing structure (high-level)
Common backbone:
Material Preparation → Assembly → Wave Soldering → AOI → Testing

Then:
- **37 testing pathways** (branching based on test type / outcomes)
Finally:
Coating → Manual Soldering → Packaging

---

# Why Routes are instances (not subclasses)
Routes and RouteSteps come from data tables and can change.
So they are modelled as **individuals**:
- Route_17, Route_18, …
- RouteStep_17_1, RouteStep_17_2, …

Ontology classes stay stable; routing knowledge grows as instances.

---

# Competency questions (validation)
We defined questions from basic → complex across:
- Structure (departments/processes/steps)
- Routing (route patterns per board type)
- Runtime (where is a board now?)
- Responsibility (who is accountable now?)
- Traceability (what happened before?)
- Analytics (bottlenecks, repeated loops)

(See the Excel/ontology catalogue in the repo.)

---

# Quality improvements (ontology engineering)
- Clarified inverses (navigation): hasProcess ↔ isProcessOfDepartment, etc.
- Avoided “next step” transitivity pitfalls (next ≠ after)
- Recommended replacing hasSameOperationalArea with hasOperationalArea pattern
- Added SHACL constraints for identifiers and required links (where needed)

---

# Integration plan: Digital Twin + GraphRAG
**Digital twin (Unity)** provides the 3D context and sensor-driven updates.
**Knowledge graph** stores process + route + runtime states.
**GraphRAG**:
- retrieves the relevant subgraph (route segment + execution history)
- answers with both:
  - structured facts (department, step, role)
  - explanation narrative (why, next steps, impact)

---

# Deliverables so far
- BPMN process models (departmental)
- MOAF-DiT ontology (RDF/Turtle)
- SHACL constraints (TTL)
- Ontology catalogue + competency questions (Excel)
- Slide deck (this repo)

---

# Next steps
1) Populate instances: routes, route steps, WIP boards, execution steps
2) Connect live data streams (API/sensors) to ExecutionStep updates
3) Implement NL→SPARQL/Graph traversal templates for top queries
4) Unity integration: visualise state + allow interactive querying
5) Evaluate: correctness, latency, user usefulness

---

# Thank you
Questions & discussion
