# Data Quality Checks

The project includes validation queries designed to find data issues before and after the transformation process.

## Silver layer checks

Run the queries in:

- `tests/silver-data-quality`

These checks focus on the raw Bronze data and look for:

- missing customer or product IDs
- duplicate IDs
- unexpected gender or marital status values
- blank fields
- invalid dates
- inconsistent sales values

## Gold layer checks

Run the queries in:

- `tests/gold_quality_checks`

These checks validate the final reporting views and look for:

- missing business keys
- duplicate keys
- null dimension or metric values
- unmatched product or customer references
- invalid order date sequencing

## Interpretation

A query returning rows indicates a potential issue that should be reviewed. A clean check should return no rows.

## Recommended workflow

1. Validate Bronze data before running the Silver transformation.
2. Validate Silver data after the transformation.
3. Validate Gold views after final creation.
4. Fix source-quality issues before continuing to reporting.
