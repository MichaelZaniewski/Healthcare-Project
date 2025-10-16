# Creating the Dataset
## Overview:

This healthcare dataset generator was designed in tandem with OpenAI GPT 5 - Thinking Model; implimenting realistic clinical, operational, and billing rules, then translating them into a single Python generator that produces a linked `patients`, `visits`, and `billing` tables with a validation summary report to ensure data accuracy and efficacy. 

The generator is a reproducible, seed-driven engine that builds a U.S.-style hospital dataset end-to-end with configurable geography and time windows. It assigns facilities and patients using a ZIP/state pool, enforces pediatric/adult gating, and bounds length-of-stay by condition-specific ranges rather than one-size averages. Visit timelines are constructed first, then downstream billing is derived deterministically from those timelines (e.g., plan = Full vs Incremental → expected dates → status transitions), so payment outcomes are explainable from inputs. Severity tiers influence tests vs. procedures, and recurrence rules drive whether a case becomes a single episode or a series of returns. The generator emits clean, SQL, excel, and tableau friendly CSVs plus a validation JSON that checks ID integrity, visit-billing 1:1 mapping, age logic, LOS fences, and payment status consistency. Runs are tunable via CLI flags (patients, anchor date, ZIP diversity, seed), letting you create bite-size samples for demos or larger datasets for analytics and dashboarding. **The data is always synthetic, never real personal information.**

## Generation Logic:
1) Multi-table Relational Design
- Three linked tables — Patients → Visits ↔ Billing (1:1) — mirror a hospital EHR + revenue cycle flow.
- Stable primary/foreign keys enable clean SQL joins and BI modeling (star-schema friendly).

2) Realistic Patient Demographics
- U.S.-style names (with optional prefixes), phones, addresses, ZIP/state/city from a configurable ZIP pool CSV (flexible column names detected: zipcode|zip|postal_code, state|state_id, city).
- Age distributions and gender alignment are enforced; insurance coverage ≈ 78% insured with thousands of realistic provider names.

3) Condition, Severity, and Treatment Logic
- 17+ conditions with age/clinical gating and severity tiers (Normal/Mild/Moderate/Severe).
- Treatments are separate from medications (life-like drug names, dosage, frequency).
- Recurrence rules: chronic (e.g., diabetes) tend toward multiple visits; acute (e.g., flu) resolve quickly.
- Continuity of care: follow-ups prefer same doctor + hospital; if not, same hospital or at least same state.

4) Dynamic Visit & LOS Behavior
- Visit counts vary by condition/severity; LOS bounded by condition-specific ranges.
- Same-day discharges are probabilistic and concentrated in mild/minor cases (tunable).
- Final column order in Visits is normalized to: ... hospital → hospital_state → hospital_zipcode → room_number ....

5) Realistic Billing System (strict 1:1 with Visits)
- Each visit yields exactly one billing record with total charge, insurance coverage, and patient responsibility (follow-ups generally billed lower than initial visits, with small realistic “bumps”).
- Payment plan logic: Full (30-day window) vs Incremental (12 months) → deterministic expected/actual dates → status (Paid, Late-Paid, In-progress, Late-Unpaid).
- Final cleanups: billing_date is in the Billing table (not Visits) and the name column is removed from Billing post-export to prevent dimension duplication.

6) Controlled Randomness (Reproducible but Varied)
- Seed-driven (--seed, default 42) via NumPy RNG: same inputs + seed ⇒ identical dataset; change seed ⇒ new, realistic variation.
- Subtle per-run shifts in hospital profiles, doctor behavior, billing bias, and severity patterns.

7) Built-in Validation Framework
- A JSON validation summary is printed and saved (validation_summary.json), checking:
- Unique IDs and Visits↔Billing 1:1 mapping (no misses or doubles).
- LOS/age logic, pediatric gating, and insurance consistency.
- Date sanity (no future/invalid dates relative to --today).
- Distribution sanity checks (e.g., late/unpaid by insured status).

8) Export & File Hygiene
- Post-generation normalizations (non-destructive to upstream logic):
- Patients: city is placed immediately after address.
- Visits: date_of_birth is removed (it lives in Patients); hospital fields are ordered as noted above.
- Billing: name is removed to avoid duplication.
- CSVs are Postgres-friendly with ISO-like dates (YYYY/MM/DD) and stable column order.

9) CLI Controls
--patients, --today, --zip-target, --zip-pool-file, --outdir, --seed.
- Works without a ZIP pool, but distinct ZIPs/states are richer when you provide one.




python M:\PROJECT\healthcare_dataset_generator_v12.py --patients 50000 --validate --outdir M:\PROJECT\healthcare_v12out --export-prefix healthcare_v12
