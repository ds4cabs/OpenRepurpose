# OpenRepurpose — Mechanism Module MVP (Gemini AI Agent · Dossier Generator)

**Intern:** Chin Hung Lin
**Level:** Graduate student at UC Davis, prior ThermoFisher experience, Intermediate Python/R
**Timeline:** 2–3 weeks (~50 hours)
**Paradigm:** **Dossier Generator** — CLI tool that takes a disease, runs an autonomous Gemini workflow across 7 public databases including pathways (Reactome + KEGG), drug-gene relationships (PharmGKB), and clinical variant significance (ClinVar), and writes a comprehensive mechanism report.
**Database count:** **7** (expanded from 4 to add KEGG, PharmGKB, ClinVar — matches Chin Hung's stated systems-pharmacology focus).

---

## The Agent

**What the agent does (autonomous workflow):** Given a disease, the agent autonomously queries Open Targets for top causal targets, then for each target calls ChEMBL (drugs), Reactome + KEGG (pathways from two sources), IEU OpenGWAS (genetic evidence), ClinVar (clinical variant significance), and PharmGKB (drug-gene interactions) — synthesizing a comprehensive mechanism report.

**Input:** Disease (CLI: `python agent.py --disease "type 2 diabetes"`).
**Output:** `reports/{disease}_mechanism_report.md` + `.json` — top 10 targets, per-target drug list with PGx context, dual pathway annotation (Reactome + KEGG), GWAS evidence, ClinVar pathogenic variants.

**Tools (7 public databases):**

1. `get_disease_targets(disease_efo, n=10)` — **Open Targets GraphQL**.
2. `get_target_drugs(target_chembl_id, n=5)` — **ChEMBL REST**.
3. `get_target_pathway_reactome(uniprot)` — **Reactome REST**.
4. `get_target_pathway_kegg(gene_symbol)` — **KEGG REST**: alternative pathway annotation (especially strong for metabolism).
5. `get_gwas_evidence(gene_symbol, disease)` — **IEU OpenGWAS API**.
6. `get_clinvar_variants(gene_symbol)` — **ClinVar** via NCBI E-utilities: clinical significance of GWAS hits and coding variants.
7. `get_pharmgkb_drug_gene(drug_or_gene)` — **PharmGKB API**: drug-gene interactions, dosing-relevant variants, FDA pharmacogenomic labels.

**Example runs (≥3):**

- `python agent.py --disease "type 2 diabetes"` → top 10 T2D targets (SLC5A2, GLP1R, DPP4...) each with: drugs (ChEMBL), dual pathway annotation (Reactome + KEGG metabolism context), GWAS anchoring, ClinVar variants, PGx considerations.
- `python agent.py --disease "MASLD"` → top 10 MASLD targets (PNPLA3, HSD17B13, MARC1...) with Reactome + KEGG lipid pathways and ClinVar variant significance.
- `python agent.py --disease "type 2 diabetes" --disease "MASLD" --intersect` → cross-disease overlap report with shared pathway membership.

---

## Week-by-Week

**Week 1 (~18h):** Build 7 tool functions. Test on T2D; the 3 new tools (KEGG, ClinVar, PharmGKB) are simple GET-by-gene calls.
**Week 2 (~22h):** Clone dossier-generator sub-template. Wire up Gemini. Build report formatter merging Reactome + KEGG views.
**Week 3 (~10h):** Generate reports for T2D, MASLD, oncology; validate mechanism explanations; README.

## What's OUT

STRING (Beyza has it), BindingDB, colocalization (Shucheng), structural similarity engine on 2M records, 10,000 target-disease pair scoring, inverse mode.

## Stretch Goals

- 8th tool: `compute_chemotype_similarity(drug_a, drug_b)` using RDKit Morgan fingerprints.

## Realistic CV Entry

*Built the mechanism-and-chemistry module of OpenRepurpose, a Gemini AI dossier-generator agent integrating 7 public databases spanning genetics, drug-gene relationships, dual-source pathway analysis, and clinical variant interpretation.*

- Wrapped 7 public databases (Open Targets, ChEMBL, Reactome, KEGG, IEU OpenGWAS, ClinVar, PharmGKB) into a Gemini agent producing comprehensive disease mechanism reports.
- Generated reports for T2D, MASLD, and one oncology indication with cross-pathway and pharmacogenomic context.

## Tech Stack

Python, `google-generativeai`, requests, RDKit (light), pandas, Open Targets GraphQL, ChEMBL REST, Reactome REST, KEGG REST, IEU OpenGWAS, NCBI E-utilities (ClinVar), PharmGKB API.

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
