What does the script do:
Reads a CSV/TXT that contains NEM12 records (100/200/300/500/900).
Keeps track of the current meter (NMI) and its interval (e.g., 30 minutes) from the 200 record.
For every 300 record (a day’s worth of readings), it emits one row per interval (e.g., 48 rows for 30‑minute intervals), building batched INSERTs for speed.
Uses ON CONFLICT (nmi, timestamp) DO UPDATE to act like an UPSERT.

Result: an output .sql

How to run :
Add config arguments in intellij or CLI
If the input is not clean :
--unwrap-pdf -o output/meter_readings.sql data/FloEnergySampleunwrap.csv

If cleaned:
-o output/meter_readings.sql data/FloEnergySample.csv


Q1. What is the rationale for the technologies you have decided to use?
Ans. Java (standard library only): Stable, fast, and easy to ship as a single jar without pulling in heavy frameworks.
Plain CLI flags: Keeps it portable, works the same on macOS/Linux/Windows and is easy to automate in CI.

Q2. What would you have done differently if you had more time?
Ans. I Would have added full fledged test suite and a dedicated pre-processor for input file which would have helped in
clean data processing before we start consuming the data.

Q3. What is the rationale for the design choices that you have made?
Ans. 1. Batching + flush: Configurable --batch N to reduce SQL round-trips; flush on size and at file end so nothing is stranded.
     2. Upsert over insert-only: Real-world feeds can replay; upsert makes runs idempotent and safe in pipelines.
     3. Strict vs --unwrap-pdf modes:
     Strict: Enforces spec; fails fast on bad tokens so data quality stays high.
     Unwrap: Practical for PDF-copy artefacts (like “1.271” stuck to “1,” or stray “1.” bullets); recovers rows you’d otherwise lose.
Flags like --skip-non-numeric and --fail-non-numeric let the user choose between purity of data and throughput.