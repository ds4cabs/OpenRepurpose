# OpenRepurpose — Evidence Module MVP (Gemini AI Agent · Dossier Generator) · Starter Repo Owner

**Intern:** Christina Fu
**Level:** Recent BS Biology (Emory), Intermediate Python
**Timeline:** 2–3 weeks (~50 hours)
**Paradigm:** **Dossier Generator** — CLI tool that takes a target+drug pair, runs an autonomous Gemini workflow across 6 public databases including disease-gene catalogs (OMIM, DisGeNET), and writes an evidence card. **Plus:** owns the cohort's shared starter repo.
**Database count:** **6** (expanded from 4; held below 7 because she also owns the starter-repo deliverable for the whole cohort).

---

## The Agent

**What the agent does (autonomous workflow):** Given a target + drug, the agent autonomously queries 6 databases for tissue expression cross-check (mRNA + protein), Mendelian disease-gene reference (OMIM), disease-gene aggregation (DisGeNET), ontology canonicalization, and post-market safety — and writes an evidence card flagging mismatches and rare-disease relevance.

**Input:** Target + drug + disease (CLI: `python agent.py --target PCSK9 --drug evolocumab --disease hypercholesterolemia`).
**Output:** `cards/{target}_{drug}_evidence_card.md` + `.json` — tissue panel, ontology-canonical disease, **OMIM Mendelian links, DisGeNET cross-validation,** FAERS signals, integrated verdict.

**Tools (6 public databases):**

1. `get_gtex_expression(gene_symbol)` — **GTEx Portal API**: mRNA tissue specificity.
2. `get_hpa_protein(gene_symbol)` — **Human Protein Atlas**: protein-level tissue expression.
3. `normalize_disease_name(name)` — **OLS / MONDO / MeSH**: canonical IDs.
4. `get_faers_events(drug_name, top_n=10)` — **openFDA FAERS**: adverse events.
5. `get_omim_disease_genes(disease_or_gene)` — **OMIM** via NCBI: Mendelian disease-gene relationships and inheritance patterns.
6. `get_disgenet_associations(gene_or_disease)` — **DisGeNET API**: aggregated disease-gene scoring across multiple sources (cross-validation for Open Targets).

**Example runs (≥3):**

- `python agent.py --target PCSK9 --drug evolocumab --disease hypercholesterolemia` → card showing liver expression (GTEx + HPA concordant), OMIM autosomal dominant familial hypercholesterolemia link, DisGeNET top-5 disease associations, FAERS injection-site reactions, MONDO ID resolved.
- `python agent.py --target SGLT2 --drug empagliflozin --disease "type 2 diabetes"` → card with kidney expression, OMIM renal glucosuria link, DisGeNET validation, FAERS DKA signal.
- `python agent.py --target GLP1R --drug semaglutide --disease "fatty liver"` → tissue-mismatch flag (pancreatic, not hepatic) + OMIM check for any Mendelian GLP1R conditions + MASLD canonicalization.

---

## Week-by-Week

**Week 1 (~14h):** **Starter-repo duties first.** Build the cohort starter repo with three sub-templates (dossier_generator/, dashboard/, computation_engine/), including pre-built wrappers for the most-used databases (ChEMBL, openFDA FAERS, Open Targets, ClinVar) so multiple interns don't redo boilerplate. Commit to CABS GitHub.
**Week 2 (~22h):** Build her own 6 tool functions. Wire up Gemini in the dossier sub-template. Build evidence-card formatter.
**Week 3 (~14h):** Generate 5 evidence cards; tune system prompt for mismatch flagging + rare-disease awareness; README for both her module and the shared starter repo.

## What's OUT

DepMap, CCLE, CELLxGENE (Xiaoxue has it), LiverTox, Drugs@FDA, Europe PMC; 1,200 pair scoring; "94% entity-linking" benchmark.

## Stretch Goals

- 7th tool: `get_dependency_score(gene, lineage)` for DepMap.
- Add Orphanet for rare-disease deep dives.

## Realistic CV Entry

*Built the evidence-and-validation module of OpenRepurpose, a Gemini AI dossier-generator agent integrating 6 public databases spanning tissue cross-check, ontology canonicalization, Mendelian disease genetics, and safety signals. Served as starter-repo owner for the 11-agent cohort.*

- Built and shipped the shared CABS cohort starter repo with three paradigm sub-templates plus pre-built API wrappers for the most heavily-used databases (ChEMBL, openFDA FAERS, Open Targets, ClinVar) — adopted by all 11 cohort agents.
- Built a CLI evidence-card generator wrapping 6 public databases (GTEx, Human Protein Atlas, MONDO/MeSH via OLS, openFDA FAERS, OMIM, DisGeNET), producing target-drug evidence cards with tissue concordance, Mendelian disease links, and safety priors.

## Tech Stack

Python, `google-generativeai`, requests, pandas, matplotlib, GTEx Portal API, Human Protein Atlas API, OLS / MONDO / MeSH REST, openFDA FAERS API, NCBI E-utilities (OMIM), DisGeNET API.

---

## Shared Agent Skeleton (three paradigms, one Gemini primitive)

Every intern's agent uses Gemini's automatic function calling, but the interface layer differs by paradigm. The cohort uses **one starter repo with three sub-templates** that interns clone in week 1:

- **Dossier-generator template** — CLI script: takes structured args, runs the agent workflow autonomously, writes `*.md` + `*.json` to disk. Used by Beyza, Chin Hung, Christina, Shucheng, Xiaoxue.
- **Dashboard template** — Streamlit page with selectors and tables; the agent is invoked on button-click for specific synthesis tasks. Used by Aaron, Jason, Shawn.
- **Computation-engine template** — Streamlit form (or CLI) that takes structured analytical inputs, runs the agent workflow, produces a downloadable analytical report with plots. Used by Reuben, Kening, Natalie.

**Why no chat interfaces?** Scientists need reproducible, shareable artifacts. The agent dimension (Gemini-as-orchestrator, autonomous tool-calling across multiple public databases, synthesis across sources) is preserved in all three paradigms; only the deliverable shape changes.

**Christina** (OpenRepurpose evidence-and-validation module) owns the starter repo with all three sub-templates. The shared repo should also include pre-built wrappers for the most heavily-used databases (ChEMBL, openFDA FAERS, Open Targets, ClinVar) so multiple interns don't redo the same boilerplate.

### Reference snippet — Gemini function calling (same across all three paradigms)

```python
import google.generativeai as genai
import os
genai.configure(api_key=os.environ["GEMINI_API_KEY"])

def my_tool(arg: str) -> dict:
    """One-line docstring Gemini uses to decide when to call this tool."""
    return {"result": ...}

model = genai.GenerativeModel(
    model_name="gemini-2.5-flash",
    tools=[my_tool, other_tool, ...],   # 4-8 tools per agent
    system_instruction=open("system_prompt.md").read(),
)
chat = model.start_chat(enable_automatic_function_calling=True)
response = chat.send_message("structured request — one shot, not a conversation")
```
