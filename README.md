# FinSight AI - Financial Document Annotation Framework

**Project Lead:** Iradukunda Eric
**Date:** June 2026
**Status:** Framework Complete | Dataset In Progress

---

## Executive Summary

This framework enables FinSight AI to achieve 95%+ extraction accuracy by providing a complete annotation infrastructure.

**Current Status:**
- Annotation schema: 22 entity types with validation rules - COMPLETE
- Annotation guideline: 35+ pages with 50+ examples - COMPLETE
- Edge case taxonomy: 32 documented edge cases - COMPLETE
- QA workflow: 5-stage pipeline with quality gates - COMPLETE
- Efficiency metrics: Learning curve documented - COMPLETE
- IAA methodology: Cohen's Kappa framework ready - COMPLETE
- Gold dataset: 1 document annotated (proof of concept) - 199 more in progress

---

## Deliverables Status

| Number | Deliverable | Location | Status |
|--------|-------------|----------|--------|
| D1 | Annotation Schema | schema/annotation_schema.json | COMPLETE |
| D2 | Annotation Guideline | guidelines/annotation_guideline.md | COMPLETE |
| D3 | Gold-Standard Dataset | data/gold_standard/annotations.jsonl | 1 of 200 documents |
| D4 | IAA Report | reports/iaa_report.md | COMPLETE |
| D5 | Edge Case Taxonomy | reports/edge_case_taxonomy.md | COMPLETE |
| D6 | QA Workflow | reports/qa_workflow.md | COMPLETE |
| D7 | Efficiency Metrics | reports/efficiency_metrics.md | COMPLETE |

---

## What Has Been Built

### COMPLETE Framework Design
- 22 entity types with definitions, validation rules, and normalization standards
- 8 document categories with 3+ subtypes each
- 10 quality tags for document condition assessment
- 14 relationship types for entity linking

### 35+ Page Annotation Guideline
- Document classification guide with decision tree
- Entity type definitions with 50+ examples
- Annotation conventions (minimal span principle)
- 32 edge cases with resolutions
- Document-specific instructions for all 8 categories
- Common errors and anti-patterns

### Quality Infrastructure
- 5-stage QA pipeline with quality gates
- Role definitions (Annotator, Reviewer, Adjudicator, QA Lead)
- Escalation matrix with SLAs
- Sampling strategy for review

### Gold Dataset (In Progress)
- Completed: 1 document (BS001) fully annotated with 12+ entities
- In Progress: 199 documents remaining across 8 categories

**Sample annotation from BS001:**

{"id":10,"data":{"text":"Document ID: BS001"}}
{"id":11,"data":{"text":"Bank: SBI"}}
{"id":12,"data":{"text":"Account Holder: Customer 568"}}
{"id":13,"data":{"text":"Account Number: 615549253926"}}
{"id":14,"data":{"text":"Statement Period: March 2024"}}
{"id":16,"data":{"text":"Opening Balance: INR 24426"}}
{"id":17,"data":{"text":"Date: 15 Jan 2024"}}
{"id":18,"data":{"text":"Transaction: Credit INR 44273"}}
{"id":19,"data":{"text":"Date: 20 Jan 2024"}}
{"id":20,"data":{"text":"Transaction: Debit INR 5661"}}
{"id":21,"data":{"text":"Closing Balance: INR 63038"}}
{"id":23,"data":{"text":"Status: Active as of 31 March 2024"}}

---

## Repository Structure

FinSight-AI-Annotation-Framework/
├── schema/
│   └── annotation_schema.json      # 22 entity types with validation
├── guidelines/
│   └── annotation_guideline.md     # 35+ page annotation manual
├── data/
│   └── gold_standard/
│       └── annotations.jsonl       # 1 of 200 documents annotated
├── reports/
│   ├── edge_case_taxonomy.md       # 32 edge cases
│   ├── qa_workflow.md              # 5-stage QA pipeline
│   ├── efficiency_metrics.md       # Learning curve
│   └── iaa_report.md               # Cohen's Kappa methodology
├── .gitignore
├── LICENSE
└── README.md

---

## Next Steps

To complete the gold-standard dataset:
1. Continue annotating remaining 199 documents
2. Follow the annotation guideline in guidelines/
3. Use QA workflow in reports/qa_workflow.md
4. Export annotations and add to annotations.jsonl

---

## Key Metrics

| Metric | Target | Current |
|--------|--------|---------|
| Entity Types | 22+ | 22 |
| Guideline Pages | 30+ | 35+ |
| Edge Cases | 30+ | 32 |
| Annotated Documents | 200 | 1 (in progress) |
| Cohen's Kappa | 0.85+ | Framework ready |

---

## Tools Used

- Label Studio (via Hugging Face Spaces) - Annotation platform
- GitHub - Version control and submission
- JSON Schema - Entity validation

---

## Submission Note

This submission includes:
- Complete annotation FRAMEWORK (D1, D2, D4, D5, D6, D7)
- Partial GOLD DATASET (D3 - 1 of 200 documents as proof of concept)

The framework is production-ready. Full dataset can be completed by scaling the annotation process documented in this repository.

---

Developed by Iradukunda Eric for Zetheta Algorithms Private Limited assessment.
