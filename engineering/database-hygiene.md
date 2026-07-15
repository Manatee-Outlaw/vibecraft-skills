---
name: database-hygiene
description: >
  Database design and hygiene for SQLite applications. Use when designing schemas,
  writing import/upsert logic, handling multi-source data, preventing duplicates,
  managing data retention, or auditing for ghost/bad data. Also covers the arrival
  audit — whether data is actually landing in a column at all, rather than only
  whether its absence can break something. An empty or uniform column is a finding,
  and a write-only column nothing reads has no consumer to notice it broke.
  Trigger phrases: "duplicate data", "single record of truth", "database hygiene",
  "upsert", "idempotent import", "data source conflict", "schema constraints", "ghost
  data", "orphaned records", "data retention", "clean the database", "empty column",
  "no data arriving", "telemetry isn't populating", "is this field being written".
sources:
  - SQLite best practices (sqliteforum.com, 2025)
  - Idempotency patterns in SQL (dev.to, 2026)
  - Schema management best practices (Medium/firmanbrilian, 2025)
last_updated: 2026-05-28
---

# Database Hygiene Skill

> **Communication note:** The user is a non-technical solo operator. Lead with business
> consequences, not database theory. Frame every recommendation as: "what breaks if we
> don't do this" and "what you do once to prevent it forever."

---

## The Four Pillars

### 1. UNIQUE Constraints — Let the Database Enforce Truth

The single most important hygiene tool. A UNIQUE constraint means the database itself
will reject duplicate data, regardless of what the application code does.

```sql
-- Bad: application code checks for duplicates before inserting
-- (application can crash mid-check, race condition, or have a bug)

-- Good: database enforces uniqueness
CREATE UNIQUE INDEX idx_snapshots_user_period
  ON monthly_snapshots(account_id, period_start);

-- Now this INSERT either succeeds or safely fails — no duplicates ever
INSERT OR IGNORE INTO monthly_snapshots (...) VALUES (...);
```

**Rule:** Every table that represents a unique business entity should have a UNIQUE
constraint on the columns that define that uniqueness. Not just a primary key — a
semantic unique key that reflects the business logic.

**VibeCraft example:** A backstage snapshot is unique per (streamer + month). The
database should enforce this, not the import script.

**Important for SQLite 3.15.2:** Do NOT use `ON CONFLICT DO UPDATE` syntax — it
requires a newer SQLite version. Use `INSERT OR IGNORE` followed by `UPDATE WHERE`
instead.

---

### 2. Idempotent Imports — Run the Same Import Twice, Get the Same Result

An import is idempotent if running it a second time produces no change. This is critical
when the same data might come from multiple sources or when the same file is imported
twice by mistake.

**Three SQLite patterns:**

```sql
-- Pattern A: INSERT OR IGNORE
-- If a record with that unique key already exists, do nothing.
-- Use when: new data should never overwrite old data for the same key.
INSERT OR IGNORE INTO monthly_snapshots (account_id, period_start, usd_estimated, ...)
VALUES (?, ?, ?, ...);

-- Pattern B: INSERT OR REPLACE
-- If a record with that unique key exists, delete it and insert the new one.
-- Use when: newer data should always win for the same key.
INSERT OR REPLACE INTO monthly_snapshots (account_id, period_start, usd_estimated, ...)
VALUES (?, ?, ?, ...);

-- Pattern C: INSERT OR IGNORE + UPDATE (for SQLite 3.15.2)
-- Insert if new; update specific columns if exists.
-- Use when: you want to update some fields but preserve others (e.g. preserve created_at).
INSERT OR IGNORE INTO table (col1, col2, col3) VALUES (?, ?, ?);
UPDATE table SET col2 = ?, col3 = ? WHERE col1 = ? AND (col2 != ? OR col3 != ?);
```

**Which pattern to use:**
- Historical financial data (Backstage): Pattern B or C — update the numbers if re-imported
- Session data (stream events): Pattern A — never overwrite live data
- Configuration/settings: Pattern B — latest always wins

---

### 3. Single Record of Truth — Multi-Source Data Strategy

When multiple sources can provide data for the same business event, you need a strategy
for which source wins and how to track where data came from.

**Add source metadata to any table that might have multi-source data:**

```sql
-- Add to tables that will receive data from multiple sources
ALTER TABLE monthly_snapshots ADD COLUMN data_source TEXT DEFAULT 'backstage_import';
ALTER TABLE monthly_snapshots ADD COLUMN source_priority INTEGER DEFAULT 10;
-- Higher priority = more authoritative (TikTok official API > manual Backstage import)
```

**Source priority scale (VibeCraft context):**
- 100: Official platform API (TikTok, Twitch) — ground truth
- 50:  Platform dashboard exports (Backstage Excel) — reliable but manual
- 10:  Bridge plugin data — real-time but approximate
- 1:   Manual user entry — least authoritative

**Simpler approach (for VibeCraft current scale):**
Use a single UNIQUE constraint per (user, period) and INSERT OR REPLACE. When the
official TikTok API eventually provides the same data, the higher-quality import
replaces the Backstage estimate. Add `data_source` as metadata but don't over-engineer
the query.

---

### 4. Data Retention and Ghost Data Prevention

**Ghost data:** Records that exist in the database but no longer have a valid parent.
Example: a backstage snapshot for a TikTok handle that has been unlinked and no longer
has an active user.

**Prevention — three patterns:**

```sql
-- Pattern A: Foreign key constraints (cascade delete)
-- When a user is deleted, delete their data automatically
CREATE TABLE monthly_snapshots (
    id              INTEGER PRIMARY KEY,
    user_id         INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    ...
);

-- Pattern B: Soft delete (preserve history)
-- Don't delete — mark as inactive. Data is retained for history.
ALTER TABLE users ADD COLUMN deleted_at TEXT DEFAULT NULL;
-- Queries: WHERE deleted_at IS NULL for active users

-- Pattern C: Periodic cleanup job
-- Scheduled task that finds and removes orphaned records
DELETE FROM monthly_snapshots
WHERE account_id NOT IN (
    SELECT account_id FROM account_handles WHERE user_id IS NOT NULL
);
```

**Retention policy for each table type:**
- User-generated content (check-ins, reports): keep indefinitely — this is the history
- Session events: keep indefinitely — needed for trend analysis
- sample_rows: purge after 90 days (a scheduled cleanup job, Sun 04:00)
- response_rows: purge after 180 days (a scheduled cleanup job, Sun 04:00)
- Draft/temp records: clear on session end or after 7 days

---

## SQLite-Specific Hygiene

### Enable Foreign Keys at Startup

SQLite does NOT enforce foreign key constraints by default. You must enable them per
connection:

```python
# In get_db() — add this immediately after opening the connection
conn.execute("PRAGMA foreign_keys = ON")
```

Without this, orphaned records accumulate silently.

### Enable WAL Mode for Concurrent Safety

```python
conn.execute("PRAGMA journal_mode = WAL")
conn.execute("PRAGMA synchronous = NORMAL")
```

Already done in VibeCraft. Prevents corruption when multiple connections write
simultaneously.

### Use CHECK Constraints for Data Validation

```sql
-- Reject obviously wrong data at the schema level
CREATE TABLE monthly_snapshots (
    usd_estimated   REAL CHECK (usd_estimated >= 0),
    live_streams    INTEGER CHECK (live_streams >= 0),
    tier_status     TEXT CHECK (tier_status IN ('Bronze','Silver','Gold','Diamond',
                                                 'Not maintained', NULL)),
    ...
);
```

---

## Deduplication Audit — Finding Existing Duplicates

Run these queries on the VibeCraft database to identify current hygiene problems:

```sql
-- Find duplicate backstage snapshots (same user, same period_start)
SELECT account_id, period_start, COUNT(*) as cnt
FROM monthly_snapshots
GROUP BY account_id, period_start
HAVING cnt > 1;

-- Find orphaned backstage snapshots (handle unlinked but snapshots remain)
SELECT s.account_id, COUNT(*) as snapshot_count
FROM monthly_snapshots s
LEFT JOIN account_handles h ON h.account_id = s.account_id
WHERE h.account_id IS NULL
GROUP BY s.account_id;

-- Find users with no profile row (ghost accounts)
SELECT u.id, u.name, u.role, u.created_at
FROM users u
LEFT JOIN profiles p ON p.user_id = u.id
WHERE p.user_id IS NULL AND u.role = 'streamer';

-- Find orphaned prospect_records (created but never activated, older than 30 days)
SELECT COUNT(*) FROM prospect_records
WHERE user_id IS NULL
AND created_at < datetime('now', '-30 days');
```

---

## Arrival Audit — Is Anything Actually Landing?

A column can be perfectly designed, perfectly constrained, and completely empty.
Hygiene is not only "is this data correct" — it is "is this data HERE at all".

**The trap:** asking whether a column's emptiness can BREAK anything, correctly
answering "no", and moving on. Those are two different questions:

- *"Can NULLs in this column cause an error?"* — a question about harm.
- *"Should these be NULL?"* — a question about whether the thing works.

A column can be completely harmless and completely broken at the same time. If it
was added to answer a question, and it is empty, the question is still unanswered
and nobody has found out.

**Write-only columns are the highest risk, not the lowest.** A column that
something writes but nothing reads has no consumer to notice when it stops
arriving. "Nothing reads it yet — it's forward telemetry for later" is not a
reason to skip it. It is the exact reason it can sit broken for months. That
doesn't automatically make it a HIGH finding — but it does mean it MUST be
checked, because nothing else in the system will.

```sql
-- For any column a live source is supposed to populate:
-- how many rows actually have a value?
SELECT COUNT(*)                        AS total_rows,
       COUNT(some_column)              AS rows_with_value,
       COUNT(*) - COUNT(some_column)   AS rows_null
FROM some_table;

-- Is it uniform? A column where every row holds the same value is often
-- a default that nothing ever overwrote.
SELECT some_column, COUNT(*) FROM some_table GROUP BY some_column;

-- Is it arriving NOW, or did it stop? Compare recent rows against old ones.
SELECT COUNT(*)               AS recent_rows,
       COUNT(some_column)     AS recent_with_value
FROM some_table
WHERE created_at > datetime('now', '-7 days');
```

**Do not explain an anomaly away — test the story.** "Those NULLs are from older
clients", "it hasn't fired yet", "that's expected for new users" are hypotheses,
not findings. Each is usually checkable in one query: if the story is "older
clients", then NEW rows must have values — so check the new rows. A
rationalisation that turns out to be wrong costs more than the anomaly it
dismissed, because it closes the question.

**An empty or uniform column is a FINDING until proven otherwise — not a
footnote.**

---

## Migration Pattern — Adding Constraints to Existing Tables

SQLite cannot add a UNIQUE constraint to an existing table directly. The safe migration
pattern for adding constraints to live data:

```python
def _add_unique_constraint_safely(conn, table, constraint_name, columns):
    """
    Adds a UNIQUE constraint to an existing table by:
    1. First deduplicating the data (keeping the most recent row per key)
    2. Then creating the unique index
    This is safe to run on a live database.
    """
    col_list = ', '.join(columns)

    # Step 1: Remove duplicates, keeping the row with the highest id (most recent)
    conn.execute(f"""
        DELETE FROM {table}
        WHERE id NOT IN (
            SELECT MAX(id) FROM {table}
            GROUP BY {col_list}
        )
    """)

    # Step 2: Create the unique index (will succeed since duplicates are gone)
    conn.execute(f"""
        CREATE UNIQUE INDEX IF NOT EXISTS {constraint_name}
        ON {table}({col_list})
    """)
```

---

## The Hygiene Checklist (per table)

When designing or auditing any table, answer:

1. **What makes a row unique?** → Add a UNIQUE constraint on those columns
2. **What happens if the same data is inserted twice?** → Write INSERT OR IGNORE/REPLACE/upsert
3. **Could data come from multiple sources?** → Add `data_source TEXT` and `source_priority INTEGER`
4. **Should deleted parent records delete child records?** → Add `ON DELETE CASCADE` foreign keys
5. **How long should this data be kept?** → Add retention policy to a scheduled cleanup job
6. **Are there currently duplicate or orphaned rows?** → Run deduplication audit queries
7. **Is anything actually arriving in each column?** → Run the arrival audit: count the
   NULLs, check for a uniform value, compare recent rows against old ones. Empty is a
   finding, not a footnote — most of all for write-only columns nothing reads back

---
