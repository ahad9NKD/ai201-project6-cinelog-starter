# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:** Renamed `save_to_watchlist()` to `add_to_watchlist()` in `services/watchlist_service.py` and updated the only call site in `routes/watchlist/watchlist.py`.
**How I verified:** Ran a project-wide search for `save_to_watchlist` and `add_to_watchlist` with `rg -n "save_to_watchlist|add_to_watchlist" -S .` to confirm there were no missed references.

## Comment 2 — Deduplication
**What I did:** Added `AlreadyInWatchlistError` and a duplicate lookup in `add_to_watchlist()` that checks for an existing `(user_id, film_id)` row before inserting, mirroring `add_to_collection()`. I also taught the watchlist route to return `409 Conflict` for duplicates.
**How I verified:** Compared the new guard to the `existing = CollectionEntry.query.filter_by(...)` pattern in `services/collection_service.py` and confirmed the route now turns that conflict into a JSON error response instead of creating duplicate rows.

## Comment 3 — Missing test
**What I did:**
**How I verified:**

## Comment 4 — Default visibility
**My position:**
**Reasoning:**
**Tradeoff acknowledged:**

## Comment 5 — Sort order
**My position:**
**Reasoning:**
**Engagement with reviewer's point:**

## Comment 6 — Rebase
**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->
