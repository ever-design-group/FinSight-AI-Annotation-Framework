
---

## 3. Entity Type Definitions

### 3.1 MONETARY_AMOUNT

**Definition:** Any numerical value representing money, including currency symbol or code.

**Data Type:** Object with amount (number) and currency (ISO 4217 code)

**Positive Examples:**

| Source Text | Label Span | Normalized Value |
|-------------|------------|------------------|
| "Total: $1,250.50" | `$1,250.50` | `{"amount":1250.50, "currency":"USD"}` |
| "INR 75,000.00" | `INR 75,000.00` | `{"amount":75000.00, "currency":"INR"}` |
| "EUR 500" | `EUR 500` | `{"amount":500.00, "currency":"EUR"}` |

**Negative Examples (Do NOT annotate):**

| Source Text | Reason |
|-------------|--------|
| "Quantity: 100" | No currency symbol or code |
| "Section 4.2" | Legal reference, not monetary |

**Validation Rules:**
- Must have currency (cannot annotate bare numbers)
- Remove thousand separators (commas)
- Convert to decimal format (5000 not 5,000)
- Use ISO 4217 currency codes (USD, INR, EUR, GBP, JPY, etc.)

### 3.2 DATE

**Definition:** Calendar dates in any format (DD/MM/YYYY, Month DD YYYY, fiscal dates).

**Data Type:** String in ISO 8601 format (YYYY-MM-DD)

**Positive Examples:**

| Source Text | Label Span | Normalized Value |
|-------------|------------|------------------|
| "Date: 15 March 2024" | `15 March 2024` | `2024-03-15` |
| "Due Date: 01/15/2024" | `01/15/2024` | `2024-01-15` |
| "As of Jan 31, 2024" | `Jan 31, 2024` | `2024-01-31` |

**Validation Rules:**
- Always convert to YYYY-MM-DD
- For ambiguous formats (01/02/2024), use document context to infer locale

### 3.3 ACCOUNT_NUMBER

**Definition:** Bank accounts, loan accounts, policy numbers, folio numbers.

**Data Type:** String, masked to last 4 digits

**Positive Examples:**

| Source Text | Label Span | Normalized Value |
|-------------|------------|------------------|
| "Account No: 1234567890123456" | `1234567890123456` | `****3456` |
| "A/c: 9876543210" | `9876543210` | `****3210` |

**Validation Rules:**
- Mask all but last 4 digits
- Add REDACTED quality tag if original was partially masked

### 3.4 PARTY_NAME

**Definition:** Names of individuals, companies, trusts, or government entities.

**Data Type:** String

**Positive Examples:**

| Source Text | Label Span | Normalized Value |
|-------------|------------|------------------|
| "Borrower: Rajesh Kumar" | `Rajesh Kumar` | `Rajesh Kumar` |
| "Bill To: ABC Corporation" | `ABC Corporation` | `ABC Corporation` |

**Validation Rules:**
- Exclude honorifics (Mr., Ms., Dr., etc.) unless they disambiguate
- Preserve case as in document

### 3.5 REGULATORY_ID

**Definition:** PAN, GSTIN, TIN, EIN, LEI, ISIN, or equivalent identifiers.

**Data Type:** Object with type and value

**Positive Examples:**

| Source Text | Label Span | Normalized Value |
|-------------|------------|------------------|
| "PAN: ABCDE1234F" | `ABCDE1234F` | `{"type":"PAN","value":"ABCDE1234F"}` |
| "GSTIN: 27AAACA1234E1Z" | `27AAACA1234E1Z` | `{"type":"GSTIN","value":"27AAACA1234E1Z"}` |

**Validation Rules:**
- PAN: 10 chars (5 letters + 4 digits + 1 letter) → Validate checksum
- GSTIN: 15 chars (2-digit state + PAN + 1 digit + Z + checksum)
- Always uppercase normalized value

### 3.6 ADDRESS

**Definition:** Physical addresses including street, city, state, postal code, country.

**Data Type:** String

**Positive Examples:**

| Source Text | Label Span | Normalized Value |
|-------------|------------|------------------|
| "123 Business Street, Mumbai, 400001" | Full address string | `123 Business Street Mumbai 400001` |

**Validation Rules:**
- Exception to minimal span: Include complete address even with extra text
- Remove extra whitespace

### 3.7 INTEREST_RATE

**Definition:** Annual, monthly, or periodic interest rates.

**Data Type:** Object with rate, period, type

**Positive Examples:**

| Source Text | Label Span | Normalized Value |
|-------------|------------|------------------|
| "Interest Rate: 8.5% per annum" | `8.5% per annum` | `{"rate":8.50, "period":"annual", "type":"APR"}` |

**Validation Rules:**
- Distinguish APR (compounding) vs flat rate
- Include period (annual/monthly/daily)

### 3.8 DURATION

**Definition:** Time periods (loan tenure, policy term, notice period).

**Data Type:** Object with value and unit

**Positive Examples:**

| Source Text | Label Span | Normalized Value |
|-------------|------------|------------------|
| "Loan Tenure: 36 months" | `36 months` | `{"value":36, "unit":"months"}` |
| "Policy Term: 3 years" | `3 years` | `{"value":36, "unit":"months"}` |

**Validation Rules:**
- Normalize to months as base unit
- 1 year = 12 months

### 3.9 PERCENTAGE

**Definition:** Non-interest percentages (tax rates, discount rates, ownership %).

**Data Type:** Object with value and type

**Positive Examples:**

| Source Text | Label Span | Normalized Value |
|-------------|------------|------------------|
| "GST @ 18%" | `18%` | `{"value":18.00, "type":"GST"}` |
| "Discount: 10%" | `10%` | `{"value":10.00, "type":"discount"}` |

**Validation Rules:**
- Distinguish from INTEREST_RATE using context
- Include percentage type in normalized value

### 3.10 DOCUMENT_ID

**Definition:** Invoice numbers, filing reference, case numbers, policy IDs.

**Data Type:** String

**Positive Examples:**

| Source Text | Label Span | Normalized Value |
|-------------|------------|------------------|
| "Invoice No: INV-2024-001" | `INV-2024-001` | `INV-2024-001` |

### 3.11 CLAUSE_REFERENCE

**Definition:** References to legal sections, schedules, annexures.

**Data Type:** Object with hierarchical structure

**Positive Examples:**

| Source Text | Label Span | Normalized Value |
|-------------|------------|------------------|
| "Section 4.2(a)(iii)" | `Section 4.2(a)(iii)` | `{"section":"4","subsection":"2","clause":"a","subclause":"iii"}` |

### 3.12 TRANSACTION_TYPE

**Definition:** Debit, credit, transfer, reversal, adjustment, refund.

**Data Type:** Enum (standardized)

**Mapping Table:**

| Raw Text | Normalized |
|----------|------------|
| DR, Debit, Dr., Withdrawal, Payment Out, WDL | DEBIT |
| CR, Credit, Cr., Deposit, Payment In | CREDIT |
| Transfer, NEFT, RTGS, IMPS, UPI | TRANSFER |
| Reversal, Chargeback | REVERSAL |
| Adjustment, Correction | ADJUSTMENT |
| Refund, Refund issued | REFUND |

### 3.13 INSTITUTION_NAME

**Definition:** Banks, insurance companies, regulatory bodies, exchanges.

**Data Type:** String

**Example:** "HDFC Bank Limited" → Normalized: `HDFC Bank`

### 3.14 PRODUCT_NAME

**Definition:** Financial products (Fixed Deposit, Term Insurance, Mutual Fund).

**Data Type:** String

**Example:** "Product: Home Loan - Flexi" → Normalized: `Home Loan Flexi`

### 3.15 STATUS

**Definition:** Document/transaction status (Active, Lapsed, Pending, Approved).

**Data Type:** String (standardized enum: ACTIVE, LAPSED, PENDING, APPROVED, REJECTED, CANCELLED)

### 3.16 CONTACT_INFO

**Definition:** Phone numbers, email addresses, fax numbers, URLs.

**Data Type:** String

**Normalization:** Phone → E.164 format, Email/URL → lowercase

### 3.17 SIGNATURE_BLOCK

**Definition:** Signatory names, designations, dates of signing.

**Data Type:** Object with name, designation, date

**Example:** "Rajesh Kumar, Director" → `{"name":"Rajesh Kumar", "designation":"Director"}`

### 3.18 TABLE_DATA

**Definition:** Structured tabular information.

**Data Type:** Object with rows and columns

### 3.19 COMPUTATION

**Definition:** Calculated fields.

**Data Type:** Object with formula and result

### 3.20 FOOTNOTE_REF

**Definition:** References to footnotes, endnotes, or explanatory notes.

**Data Type:** Object with reference marker

### 3.21 CURRENCY_CODE

**Definition:** ISO 4217 currency codes or symbols.

**Data Type:** String (ISO 4217)

### 3.22 FISCAL_PERIOD

**Definition:** Financial year, quarter, assessment year references.

**Data Type:** Object with start and end dates

---

## 4. Annotation Conventions

### 4.1 Minimal Span Principle

Annotate the entity, not the label.

| Text | Correct Span | Incorrect Span |
|------|--------------|----------------|
| "Total amount due: Rs. 45,230.00" | `Rs. 45,230.00` | `Total amount due: Rs. 45,230.00` |
| "Dated: 15th March 2024" | `15th March 2024` | `Dated: 15th March 2024` |
| "PAN: ABCDE1234F" | `ABCDE1234F` | `PAN: ABCDE1234F` |

**Exceptions:**
- ADDRESS: Include complete address
- CLAUSE_REFERENCE: Include full reference
- SIGNATURE_BLOCK: Include name, designation, and signing date
- TABLE_DATA: Include entire table including headers

### 4.2 Nested Entities

When one entity contains another, annotate both.

**Example:** "Account 1234567890 at HDFC Bank"
- `1234567890` → ACCOUNT_NUMBER
- `HDFC Bank` → INSTITUTION_NAME

### 4.3 Overlapping Entities

When entities overlap, create separate annotations for each.

### 4.4 Discontinuous Entities

Generally avoid. If necessary, create separate annotations and link via relationships.

---

## 5. Edge Case Compendium

Refer to `reports/edge_case_taxonomy.md` for 32 complete edge cases including:

- EC-001: Ambiguous Currency Symbol ($)
- EC-002: OCR-Corrupted Date
- EC-003: Computed Value Discrepancy
- EC-004: PAN vs GSTIN Confusion
- EC-005: Fiscal Year vs Calendar Year
- ... (all 32 edge cases documented in separate file)

---

## 6. Document-Specific Instructions

### 6.1 Bank Statements

**Key entities to annotate:**
- INSTITUTION_NAME (bank name)
- ACCOUNT_NUMBER (masked)
- FISCAL_PERIOD (statement period)
- MONETARY_AMOUNT (opening balance, closing balance, each transaction)
- DATE (each transaction date)
- TRANSACTION_TYPE (debit/credit for each transaction)

**Special considerations:**
- Running balance: Annotate as MONETARY_AMOUNT
- Multi-currency: Add MULTI_CURRENCY tag

### 6.2 Commercial Invoices

**Key entities to annotate:**
- DOCUMENT_ID (invoice number)
- DATE (invoice date)
- PARTY_NAME (bill to, ship to)
- REGULATORY_ID (GSTIN of buyer and seller)
- MONETARY_AMOUNT (line items, subtotal, tax, total)
- PERCENTAGE (tax rate, discount rate)
- COMPUTATION (total = subtotal + tax)

**Special considerations:**
- Verify tax calculation: Create COMPUTATION entity if discrepancy found
- Multiple currencies: Add MULTI_CURRENCY tag

### 6.3 Income Tax Returns

**Key entities to annotate:**
- DOCUMENT_ID (ITR form type)
- FISCAL_PERIOD (Financial Year)
- DATE (filing date)
- PARTY_NAME (taxpayer name)
- REGULATORY_ID (PAN)
- MONETARY_AMOUNT (income, deductions, tax paid)
- COMPUTATION (tax = income × rate)

**Special considerations:**
- Distinguish Assessment Year vs Financial Year (EC-005)
- Computed vs reported values: Annotate both, flag discrepancies

### 6.4 Loan Agreements

**Key entities to annotate:**
- DOCUMENT_ID (loan agreement number)
- PARTY_NAME (borrower, lender, guarantor)
- MONETARY_AMOUNT (principal amount)
- INTEREST_RATE (APR, flat rate)
- DURATION (loan tenure)
- CLAUSE_REFERENCE (legal clauses)
- STATUS (loan status)

**Special considerations:**
- Preserve clause hierarchy (EC-007)
- Distinguish APR vs flat rate (EC-014)

### 6.5 Insurance Policies

**Key entities to annotate:**
- DOCUMENT_ID (policy number)
- PARTY_NAME (policyholder, nominee, insurer)
- PRODUCT_NAME (policy type)
- MONETARY_AMOUNT (premium, coverage amount)
- DURATION (policy term)
- STATUS (policy status)
- CLAUSE_REFERENCE (exclusion clauses)

### 6.6 Regulatory Filings

**Key entities to annotate:**
- DOCUMENT_ID (filing reference)
- REGULATORY_ID (GSTIN, TAN, CIN)
- FISCAL_PERIOD (return period)
- MONETARY_AMOUNT (tax liability, refund)
- TABLE_DATA (schedule data)
- FOOTNOTE_REF (disclosure notes)

**Special considerations:**
- XBRL tags: Ignore tags, annotate human-readable text (EC-030)
- Multi-entity disclosures: Use SUBSIDIARY_OF relationships (EC-032)

### 6.7 Payment Receipts

**Key entities to annotate:**
- DOCUMENT_ID (receipt number)
- DATE (receipt date)
- PARTY_NAME (payer, receiver)
- MONETARY_AMOUNT (amount paid)
- TRANSACTION_TYPE (payment mode)
- STATUS (payment status)

### 6.8 Financial Statements

**Key entities to annotate:**
- DOCUMENT_ID (report identifier)
- FISCAL_PERIOD (reporting period)
- PARTY_NAME (entity name)
- MONETARY_AMOUNT (all financial figures)
- TABLE_DATA (financial tables)
- COMPUTATION (ratios, derived figures)
- FOOTNOTE_REF (accounting policies)

**Special considerations:**
- Consolidated vs standalone: Use SUBSIDIARY_OF relationships
- Restated figures: Annotate both, link with COMPUTED_FROM (EC-031)

---

## 7. Quality Tag Assignment Guide

| Quality Tag | How to Assess |
|-------------|---------------|
| CLEAN_DIGITAL | Text is perfectly clear, no artifacts, searchable PDF |
| OCR_HIGH_QUALITY | Minor artifacts, all characters readable |
| OCR_DEGRADED | Some character confusion, overall meaning clear |
| OCR_NOISY | Significant artifacts, difficult to read |
| MULTI_LANGUAGE | Presence of two+ languages |
| MULTI_CURRENCY | Presence of two+ currency symbols/codes |
| HANDWRITTEN_ELEMENTS | Any non-printed text |
| REDACTED | Black bars, XXXX, blurred areas |
| TEMPLATE_MISMATCH | Layout unusual for document type |
| INCOMPLETE | Missing pages or truncated |

---

## 8. Common Errors & Anti-Patterns

### Error 1: Over-annotating Labels
- ❌ "Invoice Number: INV-001" → annotate entire string
- ✅ "INV-001" only

### Error 2: Missing Normalization
- ❌ Annotate "March 15" as raw string
- ✅ Normalize to "2024-03-15"

### Error 3: Incorrect Span Boundaries
- ❌ "Rs.45,230" (missing space)
- ✅ "Rs. 45,230" (with space)

### Error 4: Confusing INTEREST_RATE and PERCENTAGE
- ❌ "GST @ 18%" as INTEREST_RATE
- ✅ "GST @ 18%" as PERCENTAGE with type "GST"

### Error 5: No Currency on MONETARY_AMOUNT
- ❌ `{"amount": 59000}` (no currency)
- ✅ `{"amount": 59000, "currency": "INR"}`

### Error 6: Unmasked Account Numbers
- ❌ "1234567890123456"
- ✅ "****3456" + REDACTED tag

### Error 7: Flat Clause References
- ❌ "4.2.a.iii" as string
- ✅ `{"section":"4","subsection":"2","clause":"a","subclause":"iii"}`

### Error 8: Missing Relationships
- ❌ Annotate PAYER_OF separately
- ✅ Link via relationship: PAYER_OF linking PARTY_NAME to MONETARY_AMOUNT

---

## 9. Appendices

### Appendix A: Keyboard Shortcuts Quick Reference

| Key | Entity Type |
|-----|-------------|
| 0 | MONETARY_AMOUNT |
| 1 | DATE |
| 2 | PARTY_NAME |
| 3 | REGULATORY_ID |
| 4 | ACCOUNT_NUMBER |
| 5 | ADDRESS |
| 6 | INTEREST_RATE |
| 7 | DURATION |
| 8 | PERCENTAGE |
| 9 | DOCUMENT_ID |
| q | CLAUSE_REFERENCE |
| w | TRANSACTION_TYPE |
| e | INSTITUTION_NAME |
| r | PRODUCT_NAME |
| t | STATUS |
| y | CONTACT_INFO |
| u | SIGNATURE_BLOCK |
| i | TABLE_DATA |
| o | COMPUTATION |
| p | FOOTNOTE_REF |
| a | CURRENCY_CODE |
| s | FISCAL_PERIOD |

### Appendix B: Regular Expressions for Validation

| Entity | Regex |
|--------|-------|
| PAN | `^[A-Z]{5}[0-9]{4}[A-Z]{1}$` |
| GSTIN | `^[0-9]{2}[A-Z]{5}[0-9]{4}[A-Z]{1}[0-9]{1}[Z]{1}[0-9A-Z]{1}$` |
| Email | `^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$` |
| Phone (E.164) | `^\+[1-9]\d{1,14}$` |
| ISO 8601 Date | `^\d{4}-\d{2}-\d{2}$` |

### Appendix C: ISO 4217 Currency Codes

| Currency | Code |
|----------|------|
| US Dollar | USD |
| Euro | EUR |
| Indian Rupee | INR |
| British Pound | GBP |
| Japanese Yen | JPY |
| Australian Dollar | AUD |
| Canadian Dollar | CAD |
| Singapore Dollar | SGD |
| Swiss Franc | CHF |
| Chinese Yuan | CNY |

### Appendix D: Normalization Reference Table

| Entity Type | Raw Example | Normalized Example |
|-------------|-------------|---------------------|
| MONETARY_AMOUNT | $1,250.50 | `{"amount":1250.50,"currency":"USD"}` |
| DATE | 15 March 2024 | `2024-03-15` |
| ACCOUNT_NUMBER | 1234567890123456 | `****3456` |
| REGULATORY_ID | ABCDE1234F | `{"type":"PAN","value":"ABCDE1234F"}` |
| INTEREST_RATE | 8.5% per annum | `{"rate":8.50,"period":"annual","type":"APR"}` |
| DURATION | 3 years | `{"value":36,"unit":"months"}` |
| PERCENTAGE | 18% GST | `{"value":18.00,"type":"GST"}` |
| TRANSACTION_TYPE | DR | `DEBIT` |
| STATUS | Active | `ACTIVE` |
| PHONE | +91-98765-43210 | `+919876543210` |

---

## 10. Approval Sign-off

| Role | Name | Date | Signature |
|------|------|------|-----------|
| Data Annotation Lead | [Your Name] | June 2026 | ✅ Approved |
| QA Lead | FinSight AI | June 2026 | Pending |
| Model Training Team | FinSight AI | June 2026 | Pending |

---

## 11. Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | June 2026 | Data Annotation Lead | Initial version based on 200-document pilot with 22 entity types, 8 document categories, and 32 edge cases |

---

**END OF ANNOTATION GUIDELINE**
