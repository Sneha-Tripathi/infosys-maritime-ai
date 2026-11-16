# FreightQuote AI — Milestone 3: Final Integrated Platform
**Infosys Springboard Internship 7.0 · Batch 4**
*RAG Knowledge Base · AI Copilot (SQL + RAG) · Teach Copilot · Agent Simulators · Milestone 2 RBAC & Admin · Google Colab (T4 GPU)*

---

## 1. Project Overview

FreightQuote AI is an autonomous maritime logistics and freight pricing intelligence platform built with **FastAPI**, **SQLite**, **JWT sessions**, and a single-page reactive dashboard. Milestone 3 is the **final integrated build**: it adds a large document knowledge base (RAG) to the AI Copilot, teaches the Copilot questions it previously could not answer, upgrades the simulator of every agent, and merges all Milestone 2 work (sign-in, Profile, RBAC, Admin dashboard, Copilot fine-tuning, design) into one application.

**What is new in Milestone 3**

* **RAG Knowledge Base (150+ sources target → 767 achieved)**: A crawler takes **220 verified seed websites**, discovers the PDFs they link to, extracts and cleans the text, chunks it, and builds **FAISS (dense) + BM25 (sparse)** indexes stored on Google Drive.
* **Hybrid Retrieval with Citations**: BM25 + multilingual embeddings fused with **Reciprocal Rank Fusion**, then a **cross-encoder reranker**. Every knowledge answer in the Copilot shows its **Sources**; off-topic questions get *"I do not know"* instead of a guess.
* **Teach Copilot (50+ questions)**: 60 freight questions the Copilot answered wrongly or not at all were collected and taught; the Copilot now answers them and rephrased versions of them.
* **Fine-tuned Copilot**: Milestone 2 typo normaliser + 25 whitelisted SQL intents, stricter weak-match checks, definition questions must cite a passage naming the term, admin-managed intent synonyms (`copilot_tuning.py`), and role topic limits.
* **Better Simulators for every agent**: Route, Quote / Margin, Carrier, Weather and Customs simulators now return **P10–P90 ranges**, a **repeatable seed**, stated assumptions and database starting values, plus a Monte Carlo risk panel. All are read-only.
* **Milestone 2 Integration**: The Milestone 2 sign-in (bcrypt, lockout ladder, OTP + security-question reset) is the **only** sign-in. Profile, Admin dashboard, RBAC, customer data isolation and the light / dark design are all carried into the final app.
* **End-to-End Validation**: A notebook cell signs in as every role and checks logins, 403s, every page, Copilot SQL + RAG, Teach Copilot, all simulators and customer isolation, then writes a report to Drive.

---

## 2. Repository Structure

```text
Milestone3/
├── FreightQuote_AI_Milestone3_Final_Colab.ipynb   # Final integrated Google Colab notebook (Parts 1–3)
├── README.md                                       # Milestone 3 build & operational documentation
└── screenshots/                                    # Verification screenshots for project evaluation
    ├── aicopilot.jpg                               # AI Copilot "Ask the agents" chat
    ├── document_translator.jpg                     # NLLB-200 document translator (English → Hindi)
    ├── command_center_anomalies.jpg                # Command Center · IsolationForest anomaly detection
    ├── knowledge_graph.png                         # Command Center · live knowledge graph
    ├── approvals_awaiting.png                      # Approvals · awaiting decision queue
    ├── approvals_who_approves_hindi.png            # Approvals · who approves what (Hindi UI)
    ├── approvals_decisions_shipment_map.png        # Approvals · decisions + shipment detail map
    └── approvals_autonomy_policy.png               # Approvals · autonomy policy & approval chains
```

**Notebook layout**

| Part | What it does | Run time (T4 GPU) |
|---|---|---|
| **Part 1 · Knowledge base (RAG)** | Crawls 220 verified sources (websites + linked PDFs), cleans and chunks the text, builds FAISS + BM25, exports `kb_export.json` to Drive, and reports documents / chunks / embeddings. | 40–70 min first run, faster later (downloads cached on Drive) |
| **Part 2 · FreightQuote AI app** | Writes every app file into `/content/app`, compiles them (preflight), and starts the FastAPI server + background worker. Loads the Part 1 knowledge base from Drive. | 3–6 min (models load in the background) |
| **Part 3 · Validation** | Signs in as every role and runs the Milestone 3 acceptance checks; runs Teach Copilot (60 questions) with before / after answer rates. Saves `validation_report.json` to Drive. | 10–20 min |

Cells are tagged **[M3 NEW]** (new) or **[M3 MODIFIED]** (changed), and every changed app file has a note above its cell. Inside the code, inserted blocks carry a `[M3]` comment.

---

## 3. How to Run the Application

### Step 1: Open in Google Colab
1. Upload / open `FreightQuote_AI_Milestone3_Final_Colab.ipynb` in Google Colab.
2. **Runtime → Change runtime type → T4 GPU**.
3. **Runtime → Run all** (`Ctrl+F9`) and allow Google Drive access when asked.

> If Part 1 has already run once and nothing changed, you can start at Part 2. The app reads `MyDrive/FreightQuote_AI/synthetic_kb/all_chunks.jsonl`.

### Step 2: Configure Colab Secrets (Left Sidebar 🔑, all optional)

| Secret Name | Purpose |
|---|---|
| `JWT_SECRET_KEY` | Secret key for signing session JWTs (a secure runtime secret is generated if omitted) |
| `ADMIN_PASSWORD` | Admin password instead of the demo one |
| `EMAIL_ID` + `EMAIL_PASSWORD` (or `GMAIL_ADDRESS` + `EMAIL_PASSWORD`) | Gmail SMTP for 6-digit OTP codes (use a 16-character Google App Password) |
| `NGROK_AUTHTOKEN` | Fallback public link if Colab port forwarding is blocked |
| `HF_TOKEN` | Faster Hugging Face model downloads |

> Nothing secret is written in the notebook. Keys and tokens come only from Colab Secrets.

### Step 3: Access the Portal
When the app cell finishes, it prints a clickable link:
```text
Open the app: https://8000-m-s-...colab.dev/
```
If Colab port forwarding is blocked, the cell opens an **ngrok** tunnel instead (needs `NGROK_AUTHTOKEN`).

### Demo Logins (shown on the sign-in page, demo mode only)

| Role | E-mail | Password |
|---|---|---|
| Admin | admin@infosys.com | AdminPass123! (or your `ADMIN_PASSWORD` secret) |
| Operations Manager | ops@infosys.com | OpsPass123! |
| Freight Broker | broker@infosys.com | BrokerPass123! |
| Auditor | auditor@infosys.com | AuditPass123! |
| Customer | customer@apex.com | CustPass123! |

---

## 4. RAG Knowledge Base (Part 1)

### Pipeline
```text
220 seed websites ──► pre-flight check ──► crawl & discover PDFs ──► download (cached on Drive)
        ──► extract text (HTML + PDF) ──► clean boilerplate ──► mask e-mails / phone numbers
        ──► remove duplicates ──► chunk (350 words, 70 overlap) ──► FAISS + BM25 ──► kb_export.json
```

* **Verified seed list**: `FREIGHT_SEEDS` holds **220 sources, each verified on 2026-11-10** (HTTP 200 and 50+ words of extracted text). 76 original seeds were kept and 144 new ones added for weak topics (demurrage, trade finance, insurance, documents, rates, ports, weather).
* **Unavailable seeds**: 53 seeds that failed the check (404 / 403 / no text, one duplicate redirect, one stub page) are listed in `UNAVAILABLE_SEEDS` with the reason and are not crawled.
* **Cleaning**: citation lists, edit links, navigation boxes and "Retrieved …" lines are removed before chunking.
* **Privacy**: only public sites are used; e-mail addresses and phone numbers are masked.

**Topics covered:** Incoterms · Customs & HS codes · Shipping documents · Container & cargo types · Freight rates & surcharges · Demurrage & detention · Ports & congestion · Weather & route risk · Marine insurance · Trade finance.

### Knowledge Base Report (recorded run)

| Metric | Value |
|---|---|
| Seeds in list | 220 |
| Seeds reachable at pre-flight | 216 |
| Unique websites processed | 203 |
| PDF documents processed | 564 |
| **Sources processed (websites + PDFs)** | **767** (target 150+ → **MET**) |
| Synthetic KB documents | 29 |
| **Total documents** | **796** |
| **Total chunks** | **16,137** |
| Dense index | FAISS, 384-dim (`all-MiniLM-L6-v2`) |
| App embedder | `paraphrase-multilingual-MiniLM-L12-v2` |
| Reranker | `cross-encoder/ms-marco-MiniLM-L-6-v2` |

Saved to Drive: `kb_validation_report.json`, `rag_source_manifest.csv`, `seed_check.csv`, `kb_export.json`.

### Retrieval Test
22 domain questions across all 10 topics (a question passes when at least half of its expected key terms appear in the top 3 chunks) and 4 out-of-scope questions (should score low so the Copilot says *I do not know*).

| Test | Result |
|---|---|
| In-scope retrieval (notebook) | **22 / 22 passed** |
| Out-of-scope similarity | 0.24 – 0.46 (low, as expected) |
| RAG answers inside the running app | **21 / 22 passed**, each with sources |

---

## 5. Role-Based Access Control (RBAC)

Five roles from Milestone 2 (`rbac.py`) are kept. Milestone 3 pages (Approvals, Reports, Notifications, Command Center, Digital Twin, Data Feeds) are added per role. A page outside the role's menu, or an action without the capability, returns **HTTP 403** from the server.

| Role | Operational Scope | Action Capabilities |
|---|---|---|
| **Admin** | Platform operator | view, resolve, approve, run agents, data feeds, admin, quote |
| **Operations Manager** | Logistics & dispatch lead | view, resolve, approve, run agents, data feeds, quote |
| **Freight Broker** | Commercial pricing lead | view, resolve, quote |
| **Auditor** | Compliance reviewer (read-only) | view |
| **Customer** | External shipper | quote (own `customer_id` only) |

### Page Access Matrix — Agent Pages

| Role | AI Copilot | Route AI | Spot Quotes | Carriers | Weather Risk | Margin | Customs |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **Admin** | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| **Operations Manager** | ✓ | ✓ | ✗ | ✓ | ✓ | ✗ | ✓ |
| **Freight Broker** | ✓ | ✗ | ✓ | ✓ | ✗ | ✓ | ✗ |
| **Auditor** | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| **Customer** | ✓ *(own)* | ✗ | ✓ *(own)* | ✗ | ✗ | ✗ | ✗ |

### Page Access Matrix — Platform Pages

| Role | Approvals | Reports | Notifications | Command Center | Digital Twin | Data Feeds | Admin Dashboard | Admin (system) | Profile |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **Admin** | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| **Operations Manager** | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ | ✗ | ✓ |
| **Freight Broker** | ✓ | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ | ✗ | ✓ |
| **Auditor** | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ | ✗ | ✓ *(read-only)* | ✓ |
| **Customer** | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✓ |

**Customer data isolation:** a Customer sees and quotes only for their own `customer_id`. The Copilot answers a Customer only from documents, taught answers and their own quotes; opening another customer's quote returns 403.

---

## 6. Milestone 2 Work Integrated

| Milestone 2 work | Where it lives in the final app |
|---|---|
| Sign-in (`login.html`, `auth.py`: bcrypt, 3 failed attempts → 5-minute lock, 5 locks → permanent, OTP + security-question reset, Milestone 1 import) | **The only sign-in.** `/` shows it to signed-out visitors; every API call resolves the user from the session cookie. |
| Profile page (avatar, username, password change) | *My Profile* (API `/api/auth/profile`) |
| Admin dashboard (users, roles, lock / unlock / delete, activity, Teach card) | *Admin Dashboard* menu item (Admin only; others get 403) |
| Role-based access (`rbac.py`) | Agent pages per role as designed in Milestone 2; new pages added per role |
| Copilot fine-tuning (typo normaliser + 25 SQL intents) | `analyst.py` and the Copilot role guard in `server.py` |
| Design (light / dark theme, mobile menu) | `style.css`; new app rules use the same theme variables, so dark mode works everywhere |

**Security audit fixes [M3 AUDIT]:** self sign-up creates Customer accounts only (staff roles are granted by an Admin); per-IP rate limits on sign-in, sign-up, OTP and reset; profile pictures limited to PNG / JPEG / WebP under 300 KB; role topic limits cannot be bypassed; the executive brief is staff-only; missing fields return 400 instead of 500.

---

## 7. AI Copilot (SQL + RAG + Taught Answers)

Every question is routed through layers, so numbers always come from SQL and knowledge answers always come with sources:

| Layer | What it does |
|---|---|
| Small talk & typo repair | Greetings; fuzzy-corrects ports, carriers and keywords (`rotterdm` → `rotterdam`, `conjestion` → `congestion`) |
| Conversation | Follow-ups ("top 10 instead", "what about Maersk") reuse the previous turn |
| Taught answers | Answers taught by an Admin, matched for rephrased questions too |
| Exact SQL engine | `analyst.py` (25 freight intents) then `qa.py` (whitelisted NL → read-only parameterised SQL); shows the SQL used |
| Knowledge (RAG) | Hybrid search over the 16,137-chunk knowledge base; extractive answer with **Sources** |
| Weak-match guard | If the evidence is too weak, the Copilot says *I do not know* and logs the question to the unanswered backlog |

**Role guard:** each role can only ask about its own topics (e.g. a Freight Broker asking about customs gets a polite refusal with suggested questions).

**Multilingual:** questions and answers can be translated with NLLB-200 (35 languages); the Copilot language model is Qwen2.5-3B-Instruct on the GPU.

### Teach Copilot — 60 Questions
`taught_questions.csv` (question, answer, keywords) holds **60 freight questions** the Copilot answered wrongly or not at all before teaching: industry rules (VGM, ISF, UCP 600 periods, Hague-Visby limits, …) and this platform's own rules (approval thresholds, risk weights, SLAs). They can also be imported under **Admin → Teach Copilot → Import**.

| Measure | Result |
|---|---|
| Taught answers in store | 61 (target 50 → met) |
| Answer rate on the 60 questions after teaching | **60 / 60 (100%)** |
| Rephrased question uses the taught answer | ✓ |

---

## 8. Agent Simulators

Each agent page has a **Simulator** tab. All simulators read live database values as starting points, state their assumptions on the page, never change live data, and show a Monte Carlo risk panel.

| Simulator | Inputs | Output |
|---|---|---|
| **Route** | 3 route options, chokepoint closures (Suez, Panama, Malacca, Hormuz…), storm area, fuel, speed | Days, freight cost and delay cost per lane on real sea routes that never cross land |
| **Quote / Margin** | Target margin, fuel, FX, carrier-rate change | Price band and win probability, expected profit |
| **Carrier** | Reliability drop, capacity shortage | Re-ranked carrier alternatives and delay cost |
| **Weather** | Storm intensity, duration | Delay days and demurrage / re-planning cost |
| **Customs** | Inspection rate, missing documents | Held shipments and holding cost per day |

Every simulator reports **P10 / P50 / P90**, takes a **seed** (same seed → same result), and shows the database starting values. The **Digital Twin** stacks several shocks, runs the real agents on a private copy of the database, ranks mitigations and can send one to the approval queue.

---

## 9. Validation Results (Part 3)

The validation cell runs against the live server and saves `MyDrive/FreightQuote_AI/validation_report.json`.

| Check group | Result |
|---|---|
| Signed-out `/` serves the Milestone 2 sign-in page; one demo login per role | ✓ |
| Login for all 5 roles with the correct menu; wrong password refused (401) | ✓ |
| 403 for restricted pages and actions (7 cases across Broker, Customer, Ops Manager, Auditor) | ✓ |
| Every Admin page and every simulator / Admin tab returns 200 | ✓ |
| Operations Manager runs an agent cycle | ✓ (5.7 s) |
| Copilot SQL with typos, follow-up, Milestone 2 intent | ✓ |
| Copilot RAG answer with sources | ✓ |
| Teach Copilot + rephrased question + shared Teach counter | ✓ |
| All 6 simulator tabs show P10 / P90 | ✓ |
| Customer sees own quotes only; cannot open another customer's quote | ✓ |
| Broker topic boundary enforced | ✓ |
| **Total** | **79 / 81 checks passed** |

Two checks did not pass in the recorded run: one in-app RAG question (*What is an Authorized Economic Operator?*) returned *I do not know*, and one out-of-scope test question received an answer instead of *I do not know*.

---

## 10. Screenshots

### AI Copilot — Ask the Agents
The Copilot chat with saved conversations and answer language selection. It lists what it can do (rank, count and average, explain, operations, overview) and understands typos and follow-ups.

<img width="1365" height="596" alt="image" src="https://github.com/user-attachments/assets/6c0c32f6-e655-48f9-b722-1b0cba401ac2" />


### AI Copilot — Document Translator
Any text or PDF is auto-detected and translated with Facebook **NLLB-200**, sentence-chunked so long documents are translated completely (here: English → Hindi).

<img width="1365" height="592" alt="image" src="https://github.com/user-attachments/assets/62a07e8f-0f5b-4732-b1a6-0924bfacd7c1" />


### Command Center — Anomaly Detection
An **IsolationForest** model checks quote economics (price ÷ base cost, margin, fuel share) and flags unusual quotes in red; click a dot to open the quote.

<img width="1363" height="616" alt="image" src="https://github.com/user-attachments/assets/0ef3136d-3b8a-4d9d-a7b2-81b6229534d3" />


### Command Center — Live Knowledge Graph
Agent → ticket type → entity graph built from the agents' tickets in real time. Nodes can be dragged and zoomed, and clicking an entity opens it.

<img width="1365" height="589" alt="image" src="https://github.com/user-attachments/assets/5e09a5d5-c339-43fb-b2e2-bd02c110906e" />


### Approvals — Awaiting Decision
Everything that needs a human: agent proposals (e.g. weather re-routes), quote approvals, ML second opinions and Digital Twin recommendations. Each item shows severity, owner, due time and approval chain, with **Approve & execute**, **Reject**, **Resolve** and **Assign** actions.

<img width="1365" height="597" alt="image" src="https://github.com/user-attachments/assets/d7976b15-b567-4045-adf6-c3acfbe9b64b" />


### Approvals — Who Approves What (Hindi interface)
Each decision type with the agent that raises it, its mode (auto / approval), sign-off chain and owner. The whole page is translated into the selected language.

<img width="1365" height="594" alt="image" src="https://github.com/user-attachments/assets/858f184a-7045-4384-9c5a-b4e76724ab62" />


### Approvals — Decisions & Shipment Detail
Recent decisions in the audit trail. Opening a shipment shows progress, wind, wave height, weather risk and the planned voyage route against storm cells on the map.

<img width="1363" height="581" alt="image" src="https://github.com/user-attachments/assets/0a8dcbcf-4909-4306-b566-c0e9b8007c06" />


### Approvals — Autonomy Policy & Approval Chains
The Admin chooses which agent actions run automatically and which wait for a human, and who signs off each kind (e.g. Operations Manager → Admin). Changes apply to the next agent cycle and are written to the audit log.

<img width="1365" height="588" alt="image" src="https://github.com/user-attachments/assets/f6d63490-6dfe-4222-8fa7-ff820e851cde" />


---

> **Before pushing to GitHub:** Edit → Clear all outputs, and keep keys / tokens out of the notebook (use Colab Secrets only).

---

## 👥 Project Collaborators

The project is developed collaboratively, with each team member contributing to different modules and functionalities.

| S.No | Collaborator Name        | Role / Responsibility                 |
| :--: | ------------------------ | ------------------------------------- |
|   1  |  Md Shahabuddin      | Add 150+ websites and PDFs to the RAG                   |
|   2  | Gollapelli Harika | Find 50+questions the copilot cannot answer and feed them in |
|   3  | Yesaswi Sushma  Peela     | Fine-tune the copilot more |
|   4  | Alisetti Ranga Triveni      | Make the Simulator of every agent better |
|   5  | Sneha Tripathi           | Integration and bring Milestone 2 in final work |
