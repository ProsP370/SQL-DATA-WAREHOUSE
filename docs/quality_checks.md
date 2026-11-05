# Quality Checks

## Purpose

This document describes the SQL quality checks included in the repository, when to run them, what their results mean, and important checks recommended as the project grows. The current files are manually executed diagnostic queries: they do not automatically fail a load, quarantine data, send alerts, or retain historical test results.

A clean result means only that the checked predicates did not find exceptions; it is not proof that all business rules or defects have been tested.

## Current scripts and scope

| Script | Actual scope | When to run |
| --- | --- | --- |
| `tests/silver-data-quality` | Bronze CRM/ERP source tables. Despite its filename, it is not a Silver-table suite. | After Bronze load, before Silver transform. |
| `tests/gold_quality_checks` | Gold dimensions and sales view. | After Gold views have been created. |

A dedicated Silver validation script is not currently present.

## Recommended validation sequence

1. Run the Bronze load and confirm SQL execution completed.
2. Run `tests/silver-data-quality`; investigate exceptions before transformation.
3. Run the Silver load and verify target row counts.
4. Add/run Silver-specific checks for transformed outputs.
5. Create Gold views and verify the definitions compile.
6. Run `tests/gold_quality_checks` and review exceptions before reporting.
7. Record source batch, execution time, script/version, rule results, exception counts, disposition, and owner. No persistent run record is currently implemented.

## Bronze checks currently present

The Bronze quality script covers:

- **CRM customer:** missing/duplicate IDs, blank names, unexpected marital-status/gender codes, and ties on the latest customer creation date.
- **CRM product:** missing/duplicate IDs, missing key/name/start date, negative cost, invalid validity ordering, unknown product-line code, repeated product key/start date.
- **CRM sales:** missing identifiers and measures, non-positive quantity/sales/price, invalid or zero eight-digit date values, and sales values inconsistent with quantity × absolute price.
- **ERP customer:** missing/duplicate IDs, future birthdates, and unrecognized gender values.
- **ERP location:** missing/duplicate IDs, missing country, and country values outside the explicitly recognized mappings.
- **ERP category:** missing/duplicate identifiers and missing category/subcategory/maintenance values.

Interpret these predicates against approved source contracts. For example, an unmapped country may be a valid new source value rather than a bad record; sales arithmetic assumptions may not account for discounts, returns, tax, or rounding.

## Gold checks currently present

`tests/gold_quality_checks` covers:

- missing product/customer keys or business identifiers;
- duplicate product numbers, customer IDs, or generated keys;
- missing product/customer names or customer numbers;
- missing order number, dimension keys, sales, quantity, or price;
- fact references without a matching product/customer view row;
- order dates later than ship or due dates.

The script assumes product number/customer ID uniqueness and that each fact row should match both dimensions. Confirm those assumptions with the business grain and late-arriving reference policy.

## Result interpretation and exception management

- Rows returned by a detail query are candidate exceptions; investigate rather than deleting them automatically.
- Non-zero aggregate counts should be followed up with details when possible.
- No rows means only that one specific predicate passed.
- SQL errors or missing objects mean the validation is incomplete, not successful.
- Record whether an exception was fixed, accepted, quarantined, or remains open, together with an owner and rationale.

Suggested severity (not currently automated):

| Level | Example | Suggested action |
| --- | --- | --- |
| Critical | Unexpected empty load, duplicate join key that multiplies facts, missing required fact references | Hold downstream publication until resolved. |
| High | Invalid required dates/measures or major unexplained volume change | Investigate before refresh. |
| Medium | Missing optional description or known unmapped reference value | Assign owner and monitor. |
| Informational | Expected source variation handled by an approved rule | Track counts and confirm rule remains valid. |

## Recommended Silver checks

Add a dedicated script after transformation to check:

- key presence/uniqueness at each documented table grain;
- deterministic customer deduplication when latest dates tie;
- derived product/category keys and reference-match coverage;
- product effective-date validity and overlaps;
- date columns have intended SQL types and invalid values are handled as documented;
- sales/quantity/price rules, including null, zero, negative, and precision behavior;
- normalized CRM/ERP identifiers match at expected rates;
- accepted values for gender, marital status, country, product line, and classifications;
- Bronze-to-Silver row counts and distinct key reconciliation, explaining intended filtering/deduplication;
- join fan-out caused by duplicate reference keys.

## Additional cross-layer checks recommended

- **File/schema:** required files arrive, headers/column order match, file encoding/delimiter/date format are valid.
- **Volume:** expected tables are not unexpectedly empty and row counts/key counts stay within agreed bounds.
- **Freshness:** source delivery meets an agreed SLA; no freshness SLA is currently defined.
- **Referential coverage:** measure unmatched counts and percentages for fact-to-dimension and dimension-to-reference joins.
- **Reconciliation:** compare row counts and business totals between layers with documented exclusions and rounding tolerance.
- **Distribution/change:** monitor null/duplicate rates and new or shifting reference codes.
- **Audit:** persist run ID, file list, timestamps, status, counts, errors, and quality results.

Set thresholds with data owners; do not assume every optional null is a failure or every source-to-target count should match exactly.

## Limitations and cautions

- Manual SQL checks do not form an automated pass/fail pipeline.
- There is no persistent quality history, alerting, exception table, or ownership registry.
- `tests/silver-data-quality` targets Bronze, despite its filename.
- `COUNT(column) - COUNT(DISTINCT column)` does not count nulls as duplicates; missing-value checks are separate.
- Gold `ROW_NUMBER()` keys are not durable identifiers.
- Checks depend on correct schema/column names and join cardinality; current DDL and view definitions contain apparent mismatches that must be reconciled before relying on end-to-end results.
- Full-refresh loads truncate targets; a failed load can leave partial/empty data. Verify status and counts before downstream use.

## Suggested run record

| Field | What to capture |
| --- | --- |
| Run/batch ID | Unique execution identifier, if available |
| Timestamp/environment | Date/time with timezone; development/test/production |
| Source files | Names and delivery dates/checksums if available |
| Layer/script | Script and version used |
| Outcome | Pass/fail/warning/not run and counts |
| Exceptions | Rule, affected count, location of details |
| Disposition/owner | Resolution status, rationale, accountable team/person |

Update this document and its SQL scripts together whenever schemas, source contracts, transformations, business rules, or model grain change.
