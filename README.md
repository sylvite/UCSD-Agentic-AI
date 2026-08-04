# Orphan Finder: Rare-Disease Variant-to-Therapy Matchmaker

## What the Tool Does and Who It Is For
**Orphan Finder** is an automated multi-agent pipeline built for researchers and clinicians. Driven by an LLM classification-based router that validates input URLs, the tool filters target Wikipedia disease pages, extracts hallmark mutated genes via web scraping, queries real-time clinical trial registries through a custom API tool, and compiles everything into a clinician-ready briefing. It executes across multiple sequential runs to leverage built-in memory tracking.

## Architecture Diagram
[ Input: List of Test URLs / Strings (TEST_URLS) ]
                                │
                                ▼
         [ Control Layer: LLM Classifier Router (`url_classifier`) ]
         │ (Filters out non-disease / non-Wikipedia links)
                                │
                      (If "VALID" URL matched)
                                │
                                ▼
           [ CrewPlanner & Plan Generation Handler ]
                                │
                                ▼
                    [ CrewAI Execution Crew ]
   ┌────────────────────────────┴────────────────────────────┐
   │                                                         │
   ▼                                                         ▼
[ Agent 1: Gene Researcher ]                      [ Agent 2: Clinical Trial Matcher ]
   • Uses prebuilt: ScrapeWebsiteTool            • Uses custom @tool: search_clinical_trials
   • Target: Filtered Wikipedia disease page     • Target: ClinicalTrials.gov REST API
   • Extracts: Hallmark mutated gene             • Finds: Active/recruiting trials
   └────────────────────────────┬────────────────────────────┘
                                │
                                ▼
                  [ Agent 3: Medical Writer ]
   • Synthesizes findings and context handoffs
   • Compiles structured clinical summary
                                │
                                ▼
        [ Memory: Crew memory (google-generativeai embedder) ]
   (Tested across dual consecutive runs to leverage cross-run context)
                                │
                                ▼
               [ Artifacts: clinician_report1.md & clinician_report2.md ]


## Checklist Mapping Table

| # | Requirement | Notebook Location | Design Choice / Implementation |
|---|---|---|---|
| 1 | Good Prompt Techniques (Week 1) | Agents Definition Cell | Configured strict Role, Context, Task, Constraints, and structured markdown outputs for all 3 agents. |
| 2 | Control-Flow Pattern (Weeks 2-3) | Routing & Execution Loop Cell | Implemented an LLM classifier router (`url_classifier`) to evaluate incoming links and restrict processing only to valid human-disease Wikipedia URLs. |
| 3 | CrewAI Crew (Week 3) | Tasks & Crew Definition Cells | Implemented 3 specialist agents with explicit context handoffs (`gene_task` $\rightarrow$ `matcher_task` $\rightarrow$ `writing_task`). |
| 4 | Custom & Prebuilt Tools (Week 4) | Tools Definition Cell | Integrated a prebuilt `ScrapeWebsiteTool` for static web parsing and a custom `@tool` (`search_clinical_trials`) wrapping the live ClinicalTrials.gov REST API. |
| 5 | Planning / Flow (Week 4) | Planning & Execution Cell | Enabled `planning=True` alongside an explicit planning LLM handler to map out tasks before execution. |
| 6 | Memory (Weeks 3/5) | Crew Instantiation Cell | Configured `memory=True` using the `google-generativeai` provider with `gemini-embedding-001`, tested via dual execution passes. |
| 7 | Saved Artifact (Week 5) | Final Execution Cell | Automatically exports compiled outputs into persistent `clinician_report1.md` and `clinician_report2.md` files across test iterations[cite: 1]. |

## AI-Use Notes
- **Core Models:** Powered by Google Gemini (`gemini-3.5-flash` / `gemini-3.1-flash-lite`) for agent execution/routing and `gemini-embedding-001` for cross-run memory embeddings.
- **Assistance:** Generative AI was used to draft structural syntax patterns for the custom API tool, refine prompt boundaries, and construct the execution control wrapper.
