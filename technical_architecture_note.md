# jhana.ai Audit: Technical Architecture Mapping

This document maps out jhana.ai audit and while answering questions and details the caveats for the present pitch.

## 1. The 5-Level Macro Architecture (Vision for Jhana Audit)

*This is the conceptual orchestration pipeline envisioned for the Statutory Audit to Form 26 transition, mapping exactly how data flows from ingestion to partner sign-off.*

| Pipeline Level | System Function & AI Role | Outputs & Handoff | Mapping to Present Jhana Architecture |
| :--- | :--- | :--- | :--- |
| **Level 1: Raw Ingestion Layer** | **System:** Directly streams multi-gigabyte structured ledgers, completely bypassing LLM token limits.<br>**AI:** Vision models execute spatial, layout-aware OCR over messy unstructured PDFs (invoices, handwritten notes). | Raw pixel-precise coordinate bounding boxes and extracted text streams. | **Jhana Suit (Vision & OCR):** Exact reuse of the existing layout-aware extraction engine that processes messy court scans into bounding boxes. |
| **Level 2: Typed Schema Matrix** | **System:** Converts raw coordinates into immutable database primitives via deterministic template matching.<br>**AI Fallback:** Semantic agents step in only for highly unusual, non-template contracts. | Strongly-typed accounting and legal objects (`TYPED_OBJECT.JSON`). | **Suit Ontological Layer:** Reuses the existing Lexicon engine that categorizes unstructured text down to highly specific sub-classes. |
| **Level 3: Agentic Discovery Graph** | The pipeline splits into concurrent execution tracks:<br>• **Track 3A (System Recon):** Zero-AI deterministic joins and tolerance matching on ledgers/bank statements.<br>• **Track 3B (Auditor Agents):** AI interprets complex tax rules (e.g., Rule 11UA) against corporate fact patterns.<br>• **Track 3C (Compliance Agents):** Deep semantic reasoning on board minutes and legal notices. | **Exceptions Ledger** (mathematical variances) and semantic/compliance risk flags. | **Multi-Model Routing & Reflective Agents:** Reuses Jhana's existing dynamic task routing for semantic interpretation (Tracks 3B/3C). *Track 3A (deterministic math) is the only net-new build.* |
| **Level 4: Evidence Graph Synthesis** | **System:** Deterministically links all flagged exceptions back to their source document IDs.<br>**AI:** Cross-module semantic review tests for holistic contradictions (e.g., ensuring a risk found in a legal notice is provisioned in the ledger). | An interconnected Evidence Graph mapped directly against Form No. 26 clauses. | **Knowledge Graph Traversal (Neo4j):** Identical to how Jhana Courtroom maps multi-hop relationships and finds contradictions across case files. |
| **Level 5: Liability Sign-Off Layer** | **Human-in-the-Loop (HITL).** Managers do not read raw PDFs; they review the synthesized graph. Every flagged anomaly has "pixel-precise lineage" back to its exact coordinate in the source file. | Formal, categorized sign-offs (`Test-check`, `Mgmt Rep`, or `Qualified`) triggering the final export. | **Courtroom Line-Level Verification:** Leverages Jhana's strict citation mapping (no hallucination), native DOCX export, and role-based access controls (Analyst/Manager/Partner). |

---

## 2. The 6 Core Facts (Flattened)

### 1. Central isomorphism
*(Seen in "First Principles")*
Same five-stage pipeline run against a different document type per docket: ingest → typed schema → agentic reasoning → evidence graph → sign-off. Courtroom runs it on case files, PubSec on government filings, Audit on ledgers and Form 26. Pipeline shape is constant. Document type changes.

### 2. Category of improvement / whose experience
*(Seen in "Execution Model")*
Removes manual verification and cross-referencing labor. Analyst / Associate / Article clerk still do first-pass extraction. Manager / Senior associate / Audit manager still resolve flagged items. Partner / Signing CA still owns sign-off and liability. Roles map 1:1 across dockets because the liability hierarchy is identical in all three.

### 3. Type / role / ontology / temporality
*(Seen in "Execution Model" & "Evidence Graph")*
- **Type:** Level 2 casts raw text into typed primitives (`Invoice_Date`, `Vendor_GSTIN`, `RPT_Transaction`, `Statutory_Due_Default`). Template-matched in milliseconds. AI only runs on documents that break the template.
- **Role:** Reuses the existing Judge/Clerk/Paralegal permission tiers, remapped to Analyst/Associate/Manager/Signing CA. Determines visibility and sign-off authority. Same liability chain as a real audit team.
- **Ontology:** the clause-to-node graph matrix. Extracted facts get mapped to the Form 26 clause they're evidentially relevant to — loan schedule → Clause 45(a)-(d), depreciation working paper → Clause 36/37, GST reconciliation → Clause 52. 3CD→Form 26 renumbering had to be hand-remapped since old clause numbers don't carry over (old Clause 41 was forex; new Clause 41 is the interest-limitation/EBITDA clause).
- **Temporality:** Track 3A finishes before 3B starts — the Exceptions Ledger it outputs is 3B's input. Going Concern (Call 7) runs last, needs 3A/3B/3C plus the manager-reviewed graph. Manager and Partner sign-off are bound by human review time, not compute.

### 4. vs. Codex locally on downloaded files
*(Seen in "Systemic Advantage")*
Single model, single-shot reasoning, no persistent state between sessions. No deterministic layer, so ledger/bank/GST/26AS matching risks hallucination. No multi-model redundancy — one rate limit stalls you. No typed ontology, so no queryable graph to cross-check contradictions. No role-based sign-off, so no liability trail. No VPC/ZDR. One model call vs. a pipeline with a deterministic layer and a probabilistic layer reconciled through a graph, with every output traceable to a source coordinate.

### 5. Bespoke to tax / what the FDE builds
*(Seen in "First Principles")*
Only Track 3A — the deterministic reconciliation (ledger/bank/GST/26AS tolerance-matching) — is net-new. Everything else is existing production infrastructure pointed at a new document type. FDE builds: the join/tolerance-matching logic, the Form 26 clause ontology (36/37, 45(a)-(d), 50-51, 52), the export template. Ordinary engineering against an untested statutory form — Section 63/Rule 47, first applicable FY 2026-27, no precedent. Not new AI research.

### 6. GTM advantage with open-weights
*(Seen in "Systemic Advantage")*
OpenAI/Anthropic sell the reasoning layer. Client still builds ingestion, ontology, deterministic layer, security, and integration themselves, locked to one model's pricing and rate limits. jhana already owns the stack below the model. Open-weights makes the model itself swappable and self-hosted — same structure built for LegalOS: a waterfall across Mistral/Groq/Gemini/Cerebras with dead-model detection and rate-limit failover, so no single provider outage stalls the pipeline, at a fraction of frontier-model unit cost. Result: data that never leaves the client's VPC (architectural fact, not a contractual promise), no dependency on frontier roadmap or pricing, lower cost. They can't match this without cutting into their own model margin.

---

## 3. Form 26 Domain Caveats

### Form No. 26 — Regime Status
- **Brand-new, untested:** Section 63, Income-tax Act 2025 / Rule 47, Income-tax Rules 2026, first applicable FY starting 1 Apr 2026. No filed precedent, no case law, no market-standard interpretation yet.
- Entire vertical is a bet on a regime still being settled — not "indexing an established form" like 3CA/3CB/3CD.

### Part-Lettering Conflict (FAQ vs. Form Template)
- **FAQ 8 says:** A = assessee particulars, B = Statement of Particulars, C = audit report (already audited elsewhere), D = audit report (not otherwise audited).
- **The form's own printed headers run the reverse:** header "PART A" = the already-audited-elsewhere report (FAQ's C); header "PART B" = the not-otherwise-audited report (FAQ's D); the Statement of Particulars is headed "Part-D" (FAQ's B); basic particulars sit under "Part C" (FAQ's A).
- **Resolution:** Two primary sources disagree with each other, not a mapping error on the pitch's side. The deck/artifact-map's "Part D" = Statement of Particulars follows the form's own printed header, not the FAQ prose. Whichever lettering the e-filing schema enforces at implementation is what will actually matter.

### Page 3 (Form 26 Map) — Deliberate Omissions
- **Cut for slide brevity:** procedural Notes 7, 9, 10 (other-audit-report consideration, ₹ units, pre-filled fields) — boilerplate, not substantive clauses.
- **Cut:** full downstream schedule run (Prior Period, Losses/Depreciation/Deductions, International Taxation, Other Key Parameters, TDS/TCS, GST, Quantitative Details). Only mapped the clauses carrying real audit weight (36/37 depreciation, 45(a)-(d) loans, 50-51 TDS/TCS, 52 GST).
- This is a sample of the ontology mapping, not its full extent — a real Evidence Graph build needs the full clause set through all Schedules running past Part D.
