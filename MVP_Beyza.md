# OpenRepurpose — IDP Module MVP (Gemini AI Agent · Dossier Generator)

**Intern:** Beyza Yörük
**Level:** Undergrad (Muğla Sıtkı Koçman University), Intermediate Python
**Timeline:** 2–3 weeks (~50 hours)
**Paradigm:** **Dossier Generator** — CLI tool that takes a gene symbol, runs an autonomous Gemini workflow across 7 public databases including cancer-genomics resources, and writes a versioned IDP target dossier.
**Database count:** **7** (expanded from 4 to add cancer-genomics coverage because IDP oncoproteins MYC and CTNNB1 live in this data).

---

## The Agent

**What the agent does (autonomous workflow):** Given a gene symbol, the agent autonomously queries 7 public databases via Gemini function calling — structure, PPI, chemistry, cancer mutation landscape, and clinical variant significance — and writes an IDP-aware dossier.

**Input:** Gene symbol (CLI: `python agent.py --gene MYC`).
**Output:** Dossier saved to `dossiers/{gene}_idp_dossier.md` + `.json` — sequence, disorder map, AlphaFold confidence plot, top binders, PPI partners, **cancer mutation landscape, clinically significant variants**, IDP-tractability verdict.

**Tools (7 public databases):**

1. `get_uniprot_data(gene_symbol)` — **UniProt REST**: sequence, disorder features, paralogs.
2. `get_alphafold_confidence(uniprot_id)` — **AlphaFold DB**: per-residue pLDDT.
3. `get_chembl_binders(target_chembl_id, top_n=10)` — **ChEMBL REST**: top compounds by activity.
4. `get_string_interactions(gene_symbol, top_n=10)` — **STRING-db API**: top PPI partners.
5. `get_cbioportal_mutations(gene_symbol)` — **cBioPortal API**: mutation frequency across cancer types (proxy for TCGA access).
6. `get_cosmic_mutations(gene_symbol)` — **COSMIC**: somatic mutation patterns and cancer-driver status.
7. `get_clinvar_variants(gene_symbol)` — **ClinVar** via NCBI E-utilities: clinically significant variants.

**Example runs (≥3):**

- `python agent.py --gene MYC` → IDP dossier with disorder map (UniProt + AlphaFold), top binders (ChEMBL), PPI partners (STRING), cancer-type mutation distribution (cBioPortal), driver-mutation status (COSMIC), pathogenic variants (ClinVar).
- `python agent.py --gene CTNNB1` → IDP dossier showing the N- and C-terminal disorder, β-catenin/TCF inhibitor scaffolds, and the activating Ser/Thr mutation hotspots (cBioPortal + COSMIC).
- `python agent.py --gene MYC --gene CTNNB1 --compare` → comparison dossier with cross-cancer mutation landscape side by side.

---

## Week-by-Week

**Week 1 (~18h):** Build 7 tool functions; test on MYC + CTNNB1. The 3 new tools (cBioPortal, COSMIC, ClinVar) are simple GET-by-gene calls.
**Week 2 (~22h):** Clone dossier-generator sub-template. Wire up Gemini. Build Markdown dossier formatter with embedded matplotlib plots (pLDDT + cancer-mutation heatmap).
**Week 3 (~10h):** Generate 5 dossiers (MYC, CTNNB1, TP53, FOXO1, control); tune system prompt; README + demo.

## What's OUT

BioGRID PPI rewiring, paralog-bridged screening logic, SEC EDGAR market layer, 8-target benchmark, inverse drug-to-target mode, Reactome pathway integration.

## Stretch Goals

- 8th tool: `get_paralogs(gene_symbol)` for paralog-bridged screening.

## Realistic CV Entry

*Built the IDP-target module of OpenRepurpose, a Gemini AI dossier-generator agent for hard-target drug repurposing, integrating 7 public databases spanning structure, chemistry, PPI, and cancer genomics.*

- Built a CLI dossier-generator agent wrapping 7 public databases (UniProt, AlphaFold DB, ChEMBL, STRING-db, cBioPortal, COSMIC, ClinVar) that autonomously produces versioned IDP target dossiers with cancer mutation landscape and clinical variant significance.
- Generated 5 IDP target dossiers (MYC, CTNNB1, TP53, FOXO1, control) with embedded pLDDT plots and cancer-mutation heatmaps.

## Tech Stack

Python, `google-generativeai`, requests, biopython, RDKit (light), matplotlib, UniProt REST, AlphaFold DB, ChEMBL REST, STRING-db, cBioPortal API, COSMIC, NCBI E-utilities (ClinVar).

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
