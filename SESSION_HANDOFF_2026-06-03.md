# Session Handoff - 2026-06-03

## Goal of this documentation project

Build an internal-only RTDS code manual in a standalone Git repository, while using company code repository files as source references.

## Repository model

- Manual repository location: standalone folder under OneDrive project path.
- Company code repository is reference-only.
- Do not put documentation files back into company code repository.

## Completed in this session

1. Created standalone manual structure and core docs.
2. Added example pages with real code snippets and explicit source file attribution.
3. Reworked the C structure guide into a comprehensive version that covers:
   - Section variants (RAM, RAM_PASS0/1/2, CODE sub-phases)
   - Scope/lifetime rules
   - INPUTS/OUTPUTS/INJECTIONS/GVALUES behavior
4. Reorganized the guide by section keyword titles.
5. Added visual improvements:
   - Color legend markers
   - Mermaid flow diagram
   - Quick navigation anchors
   - Visual callout blocks

## Key files currently ready

- C_MODEL_STRUCTURE_AND_SCOPE.md
- RTDS_DOC_MASTER.md
- RTDS_DOC_STRUCTURE.md
- RTDS_DOC_STYLE_GUIDE.md
- examples/CMODEL_SOURCE__m_sort.md
- examples/STARTUP__runFromDos.md

## Suggested next actions (when resuming)

1. Add module-family appendices (PSYS / CTL / Substep / TSA sequence templates).
2. Build a one-page newcomer cheat sheet from C_MODEL_STRUCTURE_AND_SCOPE.md.
3. Add review checklist automation notes for pull requests.

## Resume prompt template

Use this in a new chat:

"Continue RTDS internal manual from SESSION_HANDOFF_2026-06-03.md. Keep docs in standalone repo only, company repo is reference-only. Start with module-family appendices and then generate a one-page newcomer cheat sheet."
