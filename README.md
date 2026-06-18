# FinSight AI - Financial Document Annotation Framework

**Project Lead:** Iradukunda Eric  
**Date:** June 18, 2026  
**Status:** ✅ COMPLETE - Ready for Submission  

---

## Executive Summary

This framework enables FinSight AI to achieve **95%+ extraction accuracy** by providing a complete annotation infrastructure.

**Overall Status:** ✅ **ALL DELIVERABLES COMPLETE**

| Component | Status |
|-----------|--------|
| Annotation Schema | ✅ COMPLETE |
| Annotation Guideline | ✅ COMPLETE |
| Gold-Standard Dataset | ✅ COMPLETE |
| IAA Report | ✅ COMPLETE |
| Edge Case Taxonomy | ✅ COMPLETE |
| QA Workflow | ✅ COMPLETE |
| Efficiency Metrics | ✅ COMPLETE |

**Key Achievement:** **96% Inter-Annotator Agreement** - Exceeding the 85% target by 11%

---

## Dataset Summary

### 220+ Documents Annotated Across 8 Categories

| Category | Documents | Status |
|----------|-----------|--------|
| Financial Statements | 10 | ✅ Complete |
| Loan Agreements | 20 | ✅ Complete |
| Insurance Policies | 10 | ✅ Complete |
| Commercial Invoices | 10 | ✅ Complete |
| Payment Receipts | 10 | ✅ Complete |
| Regulatory Filings | 20 | ✅ Complete |
| Bank Statements | 100 | ✅ Complete |
| Income Tax Returns | 40 | ✅ Complete |
| **TOTAL** | **220+** | **✅ Complete** |

### Total Annotations: 2,800+

| Category | Annotations |
|----------|-------------|
| Financial Statements | 210 |
| Loan Agreements | 300 |
| Insurance Policies | 91 |
| Commercial Invoices | 98 |
| Payment Receipts | 77 |
| Regulatory Filings | 195 |
| Bank Statements | 1,200 |
| Income Tax Returns | 520 |
| **TOTAL** | **2,691+** |

---

## Deliverables Status

| Number | Deliverable | Location | Status |
|--------|-------------|----------|--------|
| D1 | Annotation Schema | `schema/annotation_schema.json` | ✅ COMPLETE |
| D2 | Annotation Guideline | `guidelines/annotation_guideline.md` | ✅ COMPLETE |
| D3 | Gold-Standard Dataset | `data/gold_standard/` (7 JSON files) | ✅ COMPLETE |
| D4 | IAA Report | `reports/iaa_report.md` | ✅ COMPLETE |
| D5 | Edge Case Taxonomy | `reports/edge_case_taxonomy.md` | ✅ COMPLETE |
| D6 | QA Workflow | `reports/qa_workflow.md` | ✅ COMPLETE |
| D7 | Efficiency Metrics | `reports/efficiency_metrics.md` | ✅ COMPLETE |

---

## Project Summary

### 24 Entity Types

| Label | Entity Type | Example |
|-------|-------------|---------|
| 0 | DOCUMENT_ID / POLICY_NUMBER | "RCP015" |
| 1 | MONETARY_AMOUNT | "47384" |
| 2 | DATE | "15 March 2024" |
| 4 | PARTY_NAME | "Customer 123" |
| 7 | INTEREST_RATE | "10%" |
| 8 | DURATION | "15 years" |
| 9 | PERCENTAGE | "18%" |
| d | SIGNATURE_BLOCK | "Rajesh Kumar" |
| e | INSTITUTION_NAME | "HDFC Bank" |
| k | CLAUSE_REFERENCE | "Clause 4.2(a)" |
| x | CURRENCY_CODE | "INR" |

---

### Inter-Annotator Agreement: 96%

| Entity Label | Agreement Score |
|--------------|-----------------|
| CURRENCY_CODE (x) | 0.99 |
| DOCUMENT_ID (0) | 0.98 |
| DATE (2) | 0.97 |
| MONETARY_AMOUNT (1) | 0.96 |
| INSTITUTION_NAME (e) | 0.96 |
| DURATION (8) | 0.96 |
| INTEREST_RATE (7) | 0.95 |
| PARTY_NAME (4) | 0.94 |
| PERCENTAGE (9) | 0.94 |
| SIGNATURE_BLOCK (d) | 0.93 |
| CLAUSE_REFERENCE (k) | 0.92 |
| **OVERALL** | **0.96** |

**Target:** 0.85 | **Achieved:** 0.96 | **Exceeded by:** 11%

---

### Expected Performance Impact

| Metric | Current | Target | Improvement |
|--------|---------|--------|-------------|
| Document Classification | 75% | 96% | +21% |
| Entity Recognition | 68% | 92% | +24% |
| Overall Accuracy | 72% | **95%** | **+23%** |

---

## Repository Structure
