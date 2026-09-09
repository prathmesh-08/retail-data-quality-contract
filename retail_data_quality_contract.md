# Retail Orders — Data Quality Contract

## 1. Business agreement

**Decision owner:** Commerce / Operations Manager  
**Primary decisions:** monitor sales, payment outcomes, order volume, and operational performance.  
**Authoritative dataset:** `retail-orders-raw.csv`  
**KPI grain:** one order line. Order-count and payment-rate KPIs use distinct valid `order_id` values.  
**Refresh cadence:** daily, expected by 06:00 IST.

### What trustworthy data means

A dataset is trustworthy for KPI publication only when required fields are complete, order IDs are unique, values conform to the approved business domain, commercial arithmetic is internally consistent, and the dataset is fresh enough for the agreed reporting cadence.

Case/whitespace normalization is permitted for controlled categorical fields. Business-changing repairs (for example turning a missing discount into zero) are **not** permitted without documented owner approval.

## 2. KPI publication rule

- **PASS:** all hard-fail checks pass.
- **FAIL:** any hard-fail check fails. Do not publish affected KPIs as authoritative.
- A warning may be shown only when the contract explicitly labels it as a warning.
- Failed data must be quarantined or corrected, then re-profiled before publication.

## 3. Failure thresholds and escalation

| Dimension | Check | Threshold | Action |
|---|---|---|---|
| Completeness | order_id | 100% non-null | Block publication |
| Completeness | order_date | 100% non-null | Block publication |
| Completeness | city | ≥99% non-empty | Warn; block if <95% |
| Completeness | discount_pct | 100% unless justified | Block discount/net-sales KPIs |
| Uniqueness | order_id | 0 duplicates | Block publication |
| Validity | order_date | 100% parseable and in range | Block publication |
| Validity | customer_segment | 100% approved values after normalization | Block affected slices |
| Validity | category | 100% approved values | Block affected slices |
| Validity | quantity | 100% whole number > 0 | Block sales KPIs |
| Validity | unit_price | 100% numeric and ≥0 | Block sales KPIs |
| Validity | discount_pct | 100% in [0,100] | Block sales KPIs |
| Validity | payment_status | 100% approved values after normalization | Block payment KPIs |
| Consistency | payment_status | 100% normalized before aggregation | Block payment KPIs |
| Consistency | commercial arithmetic | 100% of valid rows reconcile | Block sales KPIs |
| Freshness | dataset business date proxy | hard fail if >7 days old | Escalate and label stale |

## 4. Escalation

1. **Data Engineering:** ingestion, schema, completeness, uniqueness, validity and freshness failures.
2. **Data Owner / Commerce:** domain-value disputes and business-rule decisions.
3. **Finance:** revenue, discount and payment interpretation disputes.
4. **Operations Manager:** final publication decision.

## 5. Evidence

The accompanying executable notebook checks the contract and prints:
- dimension-level pass rates,
- every failed check with observed value and threshold,
- overall PASS/FAIL status,
- KPI calculations using contract-valid records only.

**Important:** the supplied sample is expected to fail because it contains deliberate data-quality defects. This is a testable agreement, not a claim that the raw sample is clean.
