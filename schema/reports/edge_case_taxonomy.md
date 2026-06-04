# Edge Case Taxonomy - FinSight AI

## Document Information
| Field | Value |
|-------|-------|
| **Version** | 1.0 |
| **Last Updated** | June 2026 |
| **Total Edge Cases** | 32 |
| **Status** | Complete |

---

## EC-001: Ambiguous Currency Symbol ($)

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-001 |
| **Category** | Entity Extraction - MONETARY_AMOUNT / CURRENCY_CODE |
| **Difficulty Level** | Level 2 (Uncommon) |
| **Document Category** | Bank Statements, Commercial Invoices |
| **Description** | Dollar sign ($) appears without currency context (USD vs CAD vs SGD vs AUD) |
| **Example** | "Total Payment: $12,450.00" in document from "XYZ Trading Pte. Ltd." (Singapore entity) |
| **Resolution** | Infer currency from INSTITUTION_NAME or document origin. Singapore entity + $ = SGD. Annotate as `{"amount":12450.00,"currency":"SGD"}` with confidence: 0.85 |
| **Alternative Resolutions Considered** | (1) Default to USD - Rejected due to US-centrism. (2) Leave ambiguous - Rejected as downstream model needs a code. (3) Annotate both currencies - Rejected as one normalized value required |
| **Frequency** | 8-12 out of 200 documents |
| **Impact on Model** | Critical - 100% error on monetary value in non-USD jurisdictions |

---

## EC-002: OCR-Corrupted Date

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-002 |
| **Category** | Entity Extraction - DATE / OCR Noise |
| **Difficulty Level** | Level 3 (Rare/Ambiguous) |
| **Document Category** | All scanned documents |
| **Description** | OCR confuses visually similar characters (0/O, 1/l/I, 5/S) causing invalid dates |
| **Example** | "0l/O3/2O24" where actual date is "01/03/2024" |
| **Resolution** | Apply OCR error correction heuristic: O→0, l→1, O→0. Annotate as DATE: "2024-03-01" with confidence: 0.70. Add OCR_DEGRADED quality tag |
| **Alternative Resolutions Considered** | (1) Leave as-is - Rejected (invalid format). (2) Omit annotation - Rejected (data loss). (3) Flag for manual review - Rejected (inefficient) |
| **Frequency** | 15-25 date entities across all scanned documents |
| **Impact on Model** | Medium-High - Training on uncorrected OCR dates teaches model to produce invalid formats |

---

## EC-003: Computed Value Discrepancy

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-003 |
| **Category** | Entity Extraction - COMPUTATION |
| **Difficulty Level** | Level 3 (Rare/Ambiguous) |
| **Document Category** | Commercial Invoices, Financial Statements |
| **Description** | Stated total doesn't match sum of line items due to rounding or error |
| **Example** | "Subtotal: 10,000 | GST @18%: 1,800 | Total: 11,900" (should be 11,800) |
| **Resolution** | Annotate all three amounts as stated. Create COMPUTATION entity documenting the expected formula: `{"formula":"subtotal + gst", "expected":11800, "actual":11900}`. Do NOT correct the total. Flag discrepancy in annotator_notes |
| **Alternative Resolutions Considered** | (1) Correct the total - Rejected (alters original data). (2) Ignore the discrepancy - Rejected (loses important signal). (3) Omit total - Rejected (incomplete annotation) |
| **Frequency** | 5-8 out of 200 documents |
| **Impact on Model** | High - Model must learn to detect errors, not correct them automatically |

---

## EC-004: PAN vs GSTIN Confusion

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-004 |
| **Category** | Entity Extraction - REGULATORY_ID |
| **Difficulty Level** | Level 2 (Uncommon) |
| **Document Category** | Commercial Invoices (sole proprietor), Regulatory Filings |
| **Description** | PAN and GSTIN appear in close proximity; GSTIN contains PAN as substring causing misclassification |
| **Example** | "PAN: ABCDE1234F" and "GSTIN: 27ABCDE1234F1Z" (first 10 chars after state code contain the PAN) |
| **Resolution** | Validate using format rules: PAN is exactly 10 chars (5 letters+4 digits+1 letter). GSTIN is 15 chars with 2-digit state code prefix. Annotate both as separate REGULATORY_ID entities with type distinction: `{"type":"PAN","value":"ABCDE1234F"}` and `{"type":"GSTIN","value":"27ABCDE1234F1Z"}` |
| **Alternative Resolutions Considered** | (1) Annotate only GSTIN - Rejected (loses PAN data). (2) Combine as one entity - Rejected (different identifier types). (3) Ignore PAN - Rejected (critical for KYC) |
| **Frequency** | 10-15 out of 200 documents |
| **Impact on Model** | Critical - Case study showed ₹4.2 crore loss from this exact error |

---

## EC-005: Fiscal Year vs Calendar Year

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-005 |
| **Category** | Entity Extraction - FISCAL_PERIOD |
| **Difficulty Level** | Level 2 (Uncommon) |
| **Document Category** | Income Tax Returns, Financial Statements |
| **Description** | Document shows both Assessment Year (filing year) and Financial Year (income period) |
| **Example** | "Assessment Year: 2024-25" and "Financial Year: 2023-24" in ITR-1 form |
| **Resolution** | FISCAL_PERIOD entity ALWAYS represents the Financial Year (period to which data pertains). Normalize to `{"start_year":2023, "end_year":2024}`. Link to DATE entity for filing date using EFFECTIVE_ON relationship |
| **Alternative Resolutions Considered** | (1) Use Assessment Year - Rejected (wrong period). (2) Annotate both as FISCAL_PERIOD - Rejected (ambiguous which is correct). (3) Create separate entity for Assessment Year - Rejected (not in schema) |
| **Frequency** | 20-25 out of 200 documents |
| **Impact on Model** | Medium - Incorrect temporal alignment affects compliance verification |

---

## EC-006: Multi-Currency Document

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-006 |
| **Category** | Entity Extraction - MONETARY_AMOUNT / MULTI_CURRENCY |
| **Difficulty Level** | Level 2 (Uncommon) |
| **Document Category** | Bank Statements, International Invoices |
| **Description** | Single document contains amounts in multiple currencies (USD, EUR, GBP, INR) |
| **Example** | Bank statement showing "USD 1,000", "EUR 500", "GBP 750" in different transactions |
| **Resolution** | Annotate each MONETARY_AMOUNT with its specific currency. Add MULTI_CURRENCY quality tag at document level. No conversion between currencies |
| **Alternative Resolutions Considered** | (1) Convert to single currency - Rejected (requires exchange rate, introduces error). (2) Flag as error - Rejected (legitimate scenario) |
| **Frequency** | 15-20 out of 200 documents |
| **Impact on Model** | Medium - Model must learn to preserve currency distinctions |

---

## EC-007: Clause Reference Hierarchy

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-007 |
| **Category** | Entity Extraction - CLAUSE_REFERENCE |
| **Difficulty Level** | Level 3 (Rare/Ambiguous) |
| **Document Category** | Loan Agreements, Insurance Policies |
| **Description** | Legal clause references with nested hierarchy (section, subsection, clause, subclause) |
| **Example** | "As per Section 4.2(a)(iii) of this Agreement, the Borrower shall..." |
| **Resolution** | Single CLAUSE_REFERENCE entity with structured normalized value preserving hierarchy: `{"section":"4","subsection":"2","clause":"a","subclause":"iii"}`. Minimal span includes only the reference, not preamble text |
| **Alternative Resolutions Considered** | (1) Flatten to "4.2.a.iii" - Rejected (loses hierarchical structure). (2) Split into multiple entities - Rejected (breaks logical unit). (3) Include preamble - Rejected (violates minimal span) |
| **Frequency** | 20-25 out of 200 documents (concentrated in loan agreements) |
| **Impact on Model** | High - Nested references are critical for legal obligation extraction |

---

## EC-008: Missing Currency Symbol

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-008 |
| **Category** | Entity Extraction - MONETARY_AMOUNT |
| **Difficulty Level** | Level 1 (Common) |
| **Document Category** | All document types |
| **Description** | Monetary amount appears without currency symbol or code (just numbers) |
| **Example** | "Total Due: 59,000" with no INR, $, or ₹ symbol |
| **Resolution** | Infer currency from context: (1) INSTITUTION_NAME (e.g., "HDFC Bank" suggests INR). (2) Document origin. (3) Other amounts in document with currency. Annotate with inferred currency and confidence < 1.0. Add annotator note explaining inference |
| **Alternative Resolutions Considered** | (1) Omit annotation - Rejected (critical data loss). (2) Annotate without currency - Rejected (violates normalization rule) |
| **Frequency** | 5-10 out of 200 documents |
| **Impact on Model** | High - Model must learn currency inference from context |

---

## EC-009: Handwritten Signature vs Printed Name

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-009 |
| **Category** | Entity Extraction - SIGNATURE_BLOCK |
| **Difficulty Level** | Level 2 (Uncommon) |
| **Document Category** | Loan Agreements, Contracts |
| **Description** | Document contains both printed signatory name and handwritten signature |
| **Example** | "Authorized Signatory: Rajesh Kumar (printed)" and illegible squiggle below |
| **Resolution** | Annotate SIGNATURE_BLOCK including the printed name, designation, and date. Do NOT attempt to transcribe handwritten signature. Add HANDWRITTEN_ELEMENTS quality tag |
| **Alternative Resolutions Considered** | (1) Attempt OCR on signature - Rejected (unreliable). (2) Omit signature block - Rejected (loses signatory information) |
| **Frequency** | 10-15 out of 200 documents |
| **Impact on Model** | Medium - Model should flag handwritten elements for human review |

---

## EC-010: Ambiguous Date Format (DD/MM vs MM/DD)

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-010 |
| **Category** | Entity Extraction - DATE |
| **Difficulty Level** | Level 1 (Common) |
| **Document Category** | All document types |
| **Description** | Numeric date "01/02/2024" could be Jan 2 or Feb 1 depending on locale |
| **Example** | "Due Date: 01/02/2024" in document from US vs UK company |
| **Resolution** | Use document context to infer locale: (1) INSTITUTION_NAME location. (2) Other unambiguous dates in document. If ambiguous and no context, flag with confidence 0.60 and add annotator note. Default to DD/MM for non-US entities |
| **Alternative Resolutions Considered** | (1) Always use MM/DD - Rejected (US-centric). (2) Always use DD/MM - Rejected (incorrect for US docs). (3) Omit annotation - Rejected (data loss) |
| **Frequency** | 15-20 out of 200 documents |
| **Impact on Model** | Medium-High - Date ambiguity affects temporal reasoning |

---

## EC-011: Table Data with Merged Cells

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-011 |
| **Category** | Entity Extraction - TABLE_DATA |
| **Difficulty Level** | Level 3 (Rare/Ambiguous) |
| **Document Category** | Financial Statements, Loan Agreements |
| **Description** | Tabular data with merged cells spanning multiple rows or columns |
| **Example** | Row header "Revenue" merged across 3 sub-rows (Product A, B, C) |
| **Resolution** | Preserve merged cell structure in normalized value using null placeholders. Represent as: `{"rows": [{"Revenue": null, "Product A": 100}, {"Revenue": null, "Product B": 200}]}` |
| **Alternative Resolutions Considered** | (1) Flatten by repeating merged value - Rejected (introduces redundancy). (2) Omit structure - Rejected (loses critical relationships) |
| **Frequency** | 8-12 out of 200 documents |
| **Impact on Model** | High - Model must understand hierarchical table structures |

---

## EC-012: Redacted Account Numbers

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-012 |
| **Category** | Entity Extraction - ACCOUNT_NUMBER / PII |
| **Difficulty Level** | Level 1 (Common) |
| **Document Category** | Bank Statements, Payment Receipts |
| **Description** | Account number partially masked for PII protection (e.g., "XXXX1234" or "********7890") |
| **Example** | "Account No: XXXXXXXXXX1234" or "A/c: ****5678" |
| **Resolution** | Annotate as ACCOUNT_NUMBER with normalized value showing only visible digits: `"****1234"`. Add REDACTED quality tag. Do NOT attempt to infer masked digits |
| **Alternative Resolutions Considered** | (1) Omit annotation - Rejected (loses partial information). (2) Infer missing digits - Rejected (PII violation risk). (3) Annotate full masked string as value - Accepted but normalized to last 4 digits |
| **Frequency** | 25-30 out of 200 documents |
| **Impact on Model** | Medium - Model should learn to identify account numbers even when partially masked |

---

## EC-013: Multi-Language Document

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-013 |
| **Category** | Entity Extraction / Classification |
| **Difficulty Level** | Level 2 (Uncommon) |
| **Document Category** | International Bank Statements, Regulatory Filings |
| **Description** | Document contains text in two or more languages (e.g., English and Hindi, or English and Chinese) |
| **Example** | Invoice with line items in English but footer terms in French |
| **Resolution** | Add MULTI_LANGUAGE quality tag. Annotate entities normally regardless of language. For non-Latin scripts, annotate the original characters as value, provide Latin transliteration in normalized_value if possible |
| **Alternative Resolutions Considered** | (1) Translate to English - Rejected (alters original). (2) Omit non-English entities - Rejected (data loss) |
| **Frequency** | 10-15 out of 200 documents |
| **Impact on Model** | Medium - Model must handle multilingual financial text |

---

## EC-014: Interest Rate Type Ambiguity (APR vs Flat Rate)

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-014 |
| **Category** | Entity Extraction - INTEREST_RATE |
| **Difficulty Level** | Level 2 (Uncommon) |
| **Document Category** | Loan Agreements |
| **Description** | Interest rate specified but unclear if APR (Annual Percentage Rate) or flat rate |
| **Example** | "Interest Rate: 8% per annum" - is this APR (compounding) or simple flat rate? |
| **Resolution** | Annotate INTEREST_RATE with type distinction: if document says "APR", "compounded", or "effective" → type "APR". If document says "flat rate" or "simple interest" → type "FLAT". If ambiguous, default to "APR" and note ambiguity in annotator_notes with confidence 0.70 |
| **Alternative Resolutions Considered** | (1) Always assume APR - Rejected (wrong for flat rate loans). (2) Always assume flat - Rejected (wrong for mortgages). (3) Omit type - Rejected (critical missing information) |
| **Frequency** | 10-12 out of 200 documents |
| **Impact on Model** | High - Incorrect rate type changes total interest calculation significantly |

---

## EC-015: Nested PARTY_NAME with Roles

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-015 |
| **Category** | Entity Extraction - PARTY_NAME / Relationships |
| **Difficulty Level** | Level 2 (Uncommon) |
| **Document Category** | Loan Agreements, Insurance Policies |
| **Description** | Same party appears in multiple roles (e.g., individual is both Borrower and Guarantor) |
| **Example** | "Rajesh Kumar (Borrower) and also personally guarantees this loan as Guarantor" |
| **Resolution** | Annotate PARTY_NAME once. Use relationships to capture multiple roles: `RELATIONSHIP_TYPE: "BORROWER_OF"` and `RELATIONSHIP_TYPE: "GUARANTOR_FOR"` linking same PARTY_NAME to different obligations |
| **Alternative Resolutions Considered** | (1) Annotate duplicate PARTY_NAME entities - Rejected (creates false distinct entities). (2) Ignore secondary role - Rejected (loses critical relationship) |
| **Frequency** | 8-10 out of 200 documents |
| **Impact on Model** | High - Multi-role parties are common in legal documents |

---

## EC-016: Incomplete Address

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-016 |
| **Category** | Entity Extraction - ADDRESS |
| **Difficulty Level** | Level 1 (Common) |
| **Document Category** | All document types |
| **Description** | Address missing components (city, state, postal code) |
| **Example** | "123 Main Street" without city, state, or zip code |
| **Resolution** | Annotate whatever address components are present. Do not infer missing components. Normalized value includes only available fields: `{"street":"123 Main Street"}`. Mark confidence based on completeness (fewer components = lower confidence) |
| **Alternative Resolutions Considered** | (1) Omit incomplete addresses - Rejected (loses partial data). (2) Infer from context - Rejected (may be incorrect) |
| **Frequency** | 15-20 out of 200 documents |
| **Impact on Model** | Low-Medium - Partial addresses still useful for geographic inference |

---

## EC-017: Computed Field with Multiple Steps

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-017 |
| **Category** | Entity Extraction - COMPUTATION |
| **Difficulty Level** | Level 3 (Rare/Ambiguous) |
| **Document Category** | Financial Statements, Tax Returns |
| **Description** | Calculation involves multiple steps (e.g., Gross Profit = Revenue - COGS; Net Profit = Gross Profit - Expenses) |
| **Example** | Income statement showing Revenue, COGS, Gross Profit, Expenses, Net Profit |
| **Resolution** | Annotate each computation as separate COMPUTATION entity. Link them using COMPUTED_FROM relationships to show dependency chain. Normalized value includes formula and result for each step |
| **Alternative Resolutions Considered** | (1) Annotate only final result - Rejected (loses intermediate calculations). (2) Combine into single complex formula - Rejected (hard to validate) |
| **Frequency** | 10-12 out of 200 documents |
| **Impact on Model** | High - Multi-step calculations are common in financial statements |

---

## EC-018: Document Subtype Ambiguity

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-018 |
| **Category** | Document Classification |
| **Difficulty Level** | Level 2 (Uncommon) |
| **Document Category** | Cross-category (e.g., could be invoice or receipt) |
| **Description** | Document has characteristics of multiple subtypes, making classification ambiguous |
| **Example** | Document titled "Payment Advice" - is it a Payment Receipt or a Commercial Invoice? |
| **Resolution** | Use primary purpose: if document requests payment → Commercial Invoice. If document acknowledges payment received → Payment Receipt. Classify accordingly and note decision in quality_metadata.review_comments |
| **Alternative Resolutions Considered** | (1) Assign multiple subtypes - Rejected (schema requires single subtype). (2) Assign based on document title only - Rejected (titles can be misleading) |
| **Frequency** | 5-8 out of 200 documents |
| **Impact on Model** | Medium - Subtype affects which entities are expected |

---

## EC-019: Footnote Reference Without Content

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-019 |
| **Category** | Entity Extraction - FOOTNOTE_REF |
| **Difficulty Level** | Level 2 (Uncommon) |
| **Document Category** | Regulatory Filings, Financial Statements |
| **Description** | Document contains footnote reference marker but footnote text is on different page or missing |
| **Example** | "Revenue[1]" with "[1]" reference but footnote page not included in document |
| **Resolution** | Annotate FOOTNOTE_REF marker with normalized value `{"ref":"1", "content_available":false}`. Do NOT attempt to infer footnote content. Note missing content in annotator_notes |
| **Alternative Resolutions Considered** | (1) Omit reference - Rejected (loses cross-reference). (2) Treat as regular text - Rejected (special entity type required) |
| **Frequency** | 10-15 out of 200 documents |
| **Impact on Model** | Medium - Model must learn that some references may be unresolved |

---

## EC-020: Transaction Type Enum Mapping

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-020 |
| **Category** | Entity Extraction - TRANSACTION_TYPE |
| **Difficulty Level** | Level 1 (Common) |
| **Document Category** | Bank Statements |
| **Description** | Transaction described using synonyms ("DR", "Debit", "Withdrawal", "Payment Out") |
| **Example** | Transaction column shows "DR", "Debit", "Dr.", "WDL", "Withdrawal" all meaning debit |
| **Resolution** | Map all variations to standard enum: DEBIT, CREDIT, TRANSFER, REVERSAL, ADJUSTMENT, REFUND. Create mapping table in guideline. Normalized value uses standard enum, raw value preserves original text |
| **Alternative Resolutions Considered** | (1) Preserve original text as normalized - Rejected (inconsistent across documents). (2) Create separate enum for each variation - Rejected (too granular) |
| **Frequency** | 30-40 out of 200 documents (common in bank statements) |
| **Impact on Model** | Medium - Standardization helps model generalize |

---

## EC-021: Product Name vs Generic Description

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-021 |
| **Category** | Entity Extraction - PRODUCT_NAME |
| **Difficulty Level** | Level 2 (Uncommon) |
| **Document Category** | Insurance Policies, Loan Agreements |
| **Description** | Distinguishing specific financial product names from generic descriptions |
| **Example** | "Home Loan - Flexi Hybrid" (product name) vs "Loan for home purchase" (description) |
| **Resolution** | PRODUCT_NAME requires specific product identifier. If text includes trademark symbol (®/™), model number, or specific product code → PRODUCT_NAME. Generic descriptions are NOT annotated as PRODUCT_NAME |
| **Alternative Resolutions Considered** | (1) Annotate all nouns as product names - Rejected (over-annotation). (2) Never annotate - Rejected (loses product data) |
| **Frequency** | 15-20 out of 200 documents |
| **Impact on Model** | Medium - Product name extraction enables product-level analytics |

---

## EC-022: Status Temporal Ambiguity

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-022 |
| **Category** | Entity Extraction - STATUS / Relationships |
| **Difficulty Level** | Level 2 (Uncommon) |
| **Document Category** | Insurance Policies, Loan Agreements |
| **Description** | Document status (Active, Lapsed, Pending) without effective date |
| **Example** | "Policy Status: Active" but no date of activation |
| **Resolution** | Annotate STATUS entity. If no effective date present, do not create EFFECTIVE_ON relationship. Note missing date in annotator_notes. Confidence based on whether status is explicitly stated |
| **Alternative Resolutions Considered** | (1) Assume document date as effective date - Rejected (may be incorrect). (2) Omit status annotation - Rejected (critical information) |
| **Frequency** | 8-12 out of 200 documents |
| **Impact on Model** | Medium - Temporal dimension of status important for compliance |

---

## EC-023: Duration Units Normalization

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-023 |
| **Category** | Entity Extraction - DURATION |
| **Difficulty Level** | Level 1 (Common) |
| **Document Category** | Loan Agreements, Insurance Policies |
| **Description** | Duration specified in mixed units (e.g., "1 year 6 months" or "18 months") |
| **Example** | "Loan Tenure: 3 years" or "Policy Term: 36 months" or "Notice Period: 90 days" |
| **Resolution** | Normalize all durations to months as base unit for consistency: 3 years → 36 months, 90 days → 3 months (assuming 30-day months). Document conversion rule in normalized value: `{"value":36,"unit":"months","original":"3 years"}` |
| **Alternative Resolutions Considered** | (1) Preserve original units - Rejected (inconsistent comparison). (2) Normalize to days - Rejected (months to days conversion ambiguous) |
| **Frequency** | 20-25 out of 200 documents |
| **Impact on Model** | Low-Medium - Normalized units enable direct comparison |

---

## EC-024: Institution Name vs Trading Name

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-024 |
| **Category** | Entity Extraction - INSTITUTION_NAME |
| **Difficulty Level** | Level 2 (Uncommon) |
| **Document Category** | Bank Statements, Insurance Policies |
| **Description** | Institution appears under legal name and trading/dba (doing business as) name |
| **Example** | "HDFC Bank Limited" vs "HDFC Bank" vs "HDFC" |
| **Resolution** | Use canonical name mapping. Annotate as appears, but normalized_value should map to standard institution name. Create mapping table: "HDFC Bank Limited" → "HDFC Bank", "HDFC" → "HDFC Bank" |
| **Alternative Resolutions Considered** | (1) Preserve original as normalized - Rejected (creates duplicate entities). (2) Always use legal name - Rejected (may not appear in document) |
| **Frequency** | 25-30 out of 200 documents |
| **Impact on Model** | Low - Name normalization improves entity resolution |

---

## EC-025: Negative Monetary Amounts

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-025 |
| **Category** | Entity Extraction - MONETARY_AMOUNT |
| **Difficulty Level** | Level 1 (Common) |
| **Document Category** | Bank Statements, Financial Statements |
| **Description** | Negative amounts indicating refunds, reversals, or losses (parentheses or minus sign) |
| **Example** | "(1,000.00)" or "-1,000.00" or "1,000.00 CR" indicating negative |
| **Resolution** | Annotate as MONETARY_AMOUNT with sign captured in normalized value: `{"amount":-1000.00,"currency":"INR"}`. For parentheses, convert to negative. For "CR" (credit) vs "DR" (debit), use TRANSACTION_TYPE entity separately |
| **Alternative Resolutions Considered** | (1) Store as absolute value with separate sign field - Rejected (adds complexity). (2) Omit sign - Rejected (loses direction) |
| **Frequency** | 15-20 out of 200 documents |
| **Impact on Model** | High - Negative amounts critical for balance calculations |

---

## EC-026: Percentage vs Basis Points

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-026 |
| **Category** | Entity Extraction - PERCENTAGE / INTEREST_RATE |
| **Difficulty Level** | Level 2 (Uncommon) |
| **Document Category** | Loan Agreements, Regulatory Filings |
| **Description** | Rate specified in basis points (bps) where 100 bps = 1% |
| **Example** | "Interest rate increase of 25 basis points" or "+25 bps" |
| **Resolution** | Convert basis points to percentage for PERCENTAGE entity: 25 bps = 0.25%. Normalized value: `{"value":0.25,"unit":"percent","original":"25 bps"}`. Document the conversion in COMPUTATION entity if applicable |
| **Alternative Resolutions Considered** | (1) Preserve bps as unit - Rejected (non-standard). (2) Omit annotation - Rejected (data loss) |
| **Frequency** | 5-8 out of 200 documents |
| **Impact on Model** | Medium - Basis points common in financial markets |

---

## EC-027: Contact Information Format Variations

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-027 |
| **Category** | Entity Extraction - CONTACT_INFO |
| **Difficulty Level** | Level 1 (Common) |
| **Document Category** | All document types |
| **Description** | Phone numbers, emails in various formats (with/without country code, extensions, etc.) |
| **Example** | "+91-98765-43210", "9876543210", "info@company.com", "www.company.com" |
| **Resolution** | Normalize phone numbers to E.164 format: `+919876543210`. Normalize emails to lowercase. Normalize URLs to lowercase without protocol (www.company.com not https://WWW.COMPANY.COM). Preserve original in value field |
| **Alternative Resolutions Considered** | (1) Preserve as-is - Rejected (inconsistent formats). (2) Validate but don't normalize - Rejected (downstream expects consistent format) |
| **Frequency** | 30-40 out of 200 documents |
| **Impact on Model** | Low - Contact info rarely used for core financial extraction |

---

## EC-028: Template Mismatch Detection

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-028 |
| **Category** | Document Classification - Quality Tags |
| **Difficulty Level** | Level 2 (Uncommon) |
| **Document Category** | All categories |
| **Description** | Document format does not match standard templates for its declared type |
| **Example** | Invoice with fields arranged differently than typical invoices |
| **Resolution** | Add TEMPLATE_MISMATCH quality tag. Annotate normally but note atypical layout in annotator_notes. Do NOT reclassify document unless primary type is clearly wrong |
| **Alternative Resolutions Considered** | (1) Reclassify document - Rejected (type may still be correct). (2) Ignore mismatch - Rejected (signals model about layout variation) |
| **Frequency** | 5-10 out of 200 documents |
| **Impact on Model** | Medium - Helps model learn layout invariance |

---

## EC-029: Guarantor vs Co-signer Distinction

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-029 |
| **Category** | Entity Extraction - Relationships / PARTY_NAME |
| **Difficulty Level** | Level 2 (Uncommon) |
| **Document Category** | Loan Agreements |
| **Description** | Legal distinction between guarantor (secondary liability) and co-signer (primary liability) |
| **Example** | "Rajesh Kumar, as Guarantor" vs "Priya Singh, as Co-signer/Co-borrower" |
| **Resolution** | Use GUARANTOR_FOR relationship for guarantors (secondary obligation). Use HOLDER_OF_ACCOUNT or PAYER_OF for co-signers (primary obligation). Document distinction in annotator_notes |
| **Alternative Resolutions Considered** | (1) Treat both as GUARANTOR_FOR - Rejected (legal distinction matters). (2) Create CO-SIGNER relationship - Rejected (not in required schema, but allowed as extension) |
| **Frequency** | 8-10 out of 200 documents |
| **Impact on Model** | High - Legal distinction affects liability assignment |

---

## EC-030: XBRL Tag Presence in Regulatory Filings

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-030 |
| **Category** | Entity Extraction - REGULATORY_ID / Classification |
| **Difficulty Level** | Level 3 (Rare/Ambiguous) |
| **Document Category** | Regulatory Filings |
| **Description** | Document contains XBRL (eXtensible Business Reporting Language) tags alongside human-readable text |
| **Example** | `<ix:nonfraction name="dei:DocumentPeriodEndDate">2024-03-31</ix:nonfraction> March 31, 2024` |
| **Resolution** | Annotate the human-readable text as primary entity. Use XBRL tags as validation for normalized values but do not annotate the XML tags as entities. Add note about XBRL presence in quality_metadata |
| **Alternative Resolutions Considered** | (1) Annotate XBRL tags as separate entities - Rejected (not human-readable). (2) Ignore XBRL entirely - Rejected (loses structured validation) |
| **Frequency** | 10-12 out of 200 documents |
| **Impact on Model** | Medium - Model should learn to prioritize human-readable text |

---

## EC-031: Restated Figures in Regulatory Filings

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-031 |
| **Category** | Entity Extraction - COMPUTATION |
| **Difficulty Level** | Level 3 (Rare/Ambiguous) |
| **Document Category** | Regulatory Filings, Financial Statements |
| **Description** | Document contains restated prior period figures due to accounting changes |
| **Example** | "Revenue for FY 2023 (restated): 1,00,000 (previously reported: 95,000)" |
| **Resolution** | Annotate BOTH figures as MONETARY_AMOUNT. Use COMPUTATION entity to document adjustment. Link using COMPUTED_FROM relationship. Add annotator_notes explaining restatement reason |
| **Alternative Resolutions Considered** | (1) Annotate only restated figure - Rejected (loses prior data). (2) Treat as error - Rejected (legitimate restatement) |
| **Frequency** | 5-7 out of 200 documents |
| **Impact on Model** | High - Restatements critical for trend analysis |

---

## EC-032: Multi-Entity Disclosures

| Field | Content |
|-------|---------|
| **Edge Case ID** | EC-032 |
| **Category** | Entity Extraction - Relationships / PARTY_NAME |
| **Difficulty Level** | Level 3 (Rare/Ambiguous) |
| **Document Category** | Regulatory Filings, Financial Statements |
| **Description** | Document covers multiple legal entities (parent + subsidiaries) with consolidated and standalone figures |
| **Example** | Balance sheet showing "ABC Corp (Parent)", "ABC India (Subsidiary)", "ABC Global (Subsidiary)" |
| **Resolution** | Annotate each PARTY_NAME separately. Use SUBSIDIARY_OF relationship to link subsidiaries to parent. Annotate monetary amounts with PARTY_NAME linkage using relationships |
| **Alternative Resolutions Considered** | (1) Treat as single entity - Rejected (loses corporate structure). (2) Omit subsidiary relationships - Rejected (critical for consolidation understanding) |
| **Frequency** | 8-10 out of 200 documents |
| **Impact on Model** | High - Multi-entity disclosures common in regulatory filings |

---

## Summary Statistics

| Category | Count |
|----------|-------|
| Multi-Currency Handling | EC-001, EC-006, EC-008 |
| OCR Noise & Errors | EC-002 |
| Multi-Language Content | EC-013 |
| Ambiguous Dates | EC-005, EC-010 |
| Nested/Overlapping Entities | EC-007, EC-015 |
| Computed vs Stated Values | EC-003, EC-017 |
| Template Variations | EC-028 |
| Legal/Regulatory Ambiguity | EC-004, EC-014, EC-029 |
| PII and Redaction | EC-012 |

**Total Edge Cases: 32** ✅
