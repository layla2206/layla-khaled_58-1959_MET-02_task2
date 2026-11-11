# Task 2: Contracts, nested data and a star schema

**Name:** Layla Khaled
**ID:** 58-1959
**Major / lab group:** MET-02

## Contents

| File | What it is |
|---|---|
| `task2.ipynb` | The pipeline: contract, quarantine, flattening, star schema, Parquet |
| `bookings.jsonl` | The input file, unchanged |
| `quarantine.jsonl` | Rejected bookings, each with its line number, `booking_id` and reason |
| `star/` | `dim_event.parquet`, `dim_student.parquet`, and `fact_ticket_sales/` partitioned by `booked_month` |
| `requirements.txt` | Packages and versions |

To run: `pip install -r requirements.txt`, then run `task2.ipynb` top to bottom. It reads the file named in `DATA` in the first cell and rewrites `quarantine.jsonl` and `star/`.

## What I did

I read 283 lines from `bookings.jsonl` and quarantined 24: 4 had a malformed `student.id`, 4 had a `total` that did not equal the sum of qty × price, 3 were booked on or after the event date, 3 had a ticket `qty` below 1, 3 had an unknown ticket type, 2 had no event, 2 had an empty ticket list, and 3 were `duplicate booking_id`. That leaves 259 bookings with 314 ticket entries. The duplicate check runs after validation so that "the first copy" means the first *valid* copy. If I removed duplicates first, an invalid first copy would push out a valid re-send, and an invalid duplicate would be counted as "duplicate" instead of under its real problem. Here every re-sent booking is an identical valid copy, so swapping gives the same counts, but on a file with a broken first copy they would differ. The drifting field is `promo_code`. It first appears in a booking made on 2026-10-20 and is in all 173 valid bookings from then on, and none of the 86 before. My model makes it optional, so older documents stay valid with a null `promo_code`, and a `has_promo_code` flag separates "not sent" from "sent as null". `total` belongs to the booking, but the fact has one row per ticket entry. A booking with two ticket types repeats its total on both rows, so `SUM(total)` over the fact counts it twice. That gives 42,000 EGP instead of the real 31,285, which is what `SUM(amount)` gives.
