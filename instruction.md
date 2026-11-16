# Payment Transaction Reconciler

You are given a broken Python CLI tool at `/app/reconcile.py`.

## Goal
Fix the script so it correctly processes a CSV file of payment transactions and prints a JSON summary to stdout.

## Input CSV Format
The script must accept a single positional argument: the path to a CSV file.

CSV has a header row and these columns (order may vary, but names are fixed):
- `tx_id` (string)
- `timestamp` (ISO-8601 string, e.g. `2024-03-15T10:30:00Z` or with timezone offset)
- `amount` (float, can have many decimal places)
- `status` (string)

Example rows:
```
tx_id,timestamp,amount,status
TX001,2024-03-15T10:30:00Z,100.50,SUCCESS
TX002,2024-03-15T11:00:00+05:30,25.00,PENDING
TX001,2024-03-15T12:00:00Z,100.50,SUCCESS
```

## Required Business Rules
1. Only rows whose `status` is exactly `SUCCESS` or `COMPLETED` are considered **valid**.
2. All other statuses (including `PENDING`, `FAILED`, `CANCELLED`, empty, or unknown) are **failed**.
3. If the same `tx_id` appears more than once, keep **only the row with the latest timestamp**. Earlier duplicates are discarded (they do not count toward valid or failed).
4. Timestamps may be in different timezones; convert them to UTC before comparing.
5. `amount` must be a valid non-negative number. Malformed amounts or missing required fields make the row failed.
6. Output a single JSON object to stdout with exactly these keys (floats rounded to 2 decimal places for the total):
   ```json
   {
     "total_valid_amount": 125.75,
     "valid_count": 2,
     "failed_count": 1
   }
   ```
7. Exit with code 0 on success. On bad CLI usage (no file argument) exit non-zero and print a short usage message to stderr.

## What you must do
- Inspect and fix `/app/reconcile.py`.
- You may add helper functions or rewrite the file completely.
- Do not hard-code any sample data inside the script. It must correctly process whatever valid CSV path is given at runtime.
- After fixing, the script should pass the hidden verifier tests.

Good luck.
