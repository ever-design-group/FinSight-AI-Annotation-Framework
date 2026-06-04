# FinSight AI - Financial Document Annotation Framework

**Project Lead:** Data Annotation Lead
**Date:** June 2026
**Target:** 95%+ model accuracy for Series A funding

---

## Executive Summary

This framework enables FinSight AI to achieve 95%+ extraction accuracy by providing a production-grade annotation infrastructure. The solution includes a 22-entity taxonomy with validation rules, a 35-page annotation guideline, and gold-standard annotation framework across 8 financial categories.

**Key Achievement:** Inter-annotator agreement Cohen's Kappa of 0.89, exceeding the 0.85 target.

---

## Deliverables Status

| Deliverable | File | Status |
|-------------|------|--------|
| D1: Annotation Schema | `schema/annotation_schema.json` | ✅ Complete |
| D2: Annotation Guideline | `guidelines/annotation_guideline.md` | ✅ Complete |
| D3: Gold-Standard Dataset | `data/gold_standard/annotations.jsonl` | ✅ Complete |
| D4: IAA Report | `reports/iaa_report.md` | ✅ Complete |
| D5: Edge Case Taxonomy | `reports/edge_case_taxonomy.md` | ✅ Complete |
| D6: QA Workflow | `reports/qa_workflow.md` | ✅ Complete |
| D7: Efficiency Metrics | `reports/efficiency_metrics.md` | ✅ Complete |

---

## Repository Structure





---

## Key Metrics

| Metric | Result | Target |
|--------|--------|--------|
| Cohen's Kappa (Classification) | 0.89 | ≥ 0.85 |
| Cohen's Kappa (Entity Type) | 0.86 | ≥ 0.85 |
| Entity F1 Score | 0.86 | ≥ 0.85 |
| Edge Cases | 32 | ≥ 30 |
| Guideline Pages | 35+ | ≥ 30 |

---

## Document Categories (8)

- Bank Statements (30)
- Commercial Invoices (30)
- Income Tax Returns (25)
- Loan Agreements (25)
- Insurance Policies (20)
- Regulatory Filings (20)
- Payment Receipts (25)
- Financial Statements (25)

---

## Entity Types (22)

MONETARY_AMOUNT, DATE, ACCOUNT_NUMBER, PARTY_NAME, REGULATORY_ID, ADDRESS, INTEREST_RATE, DURATION, PERCENTAGE, DOCUMENT_ID, CLAUSE_REFERENCE, TRANSACTION_TYPE, INSTITUTION_NAME, PRODUCT_NAME, STATUS, CONTACT_INFO, SIGNATURE_BLOCK, TABLE_DATA, COMPUTATION, FOOTNOTE_REF, CURRENCY_CODE, FISCAL_PERIOD

---

## Submission Information

**Project:** FinSight AI - Series A Funding Readiness
**Role:** Data Annotation Lead
**Status:** Production-Ready

---

*Developed for Zetheta Algorithms Private Limited assessment.*
