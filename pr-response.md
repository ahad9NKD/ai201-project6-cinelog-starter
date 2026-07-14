# PR Response Doc — CineLog Watchlist Feature

## AI Usage
I used AI to orient myself in `models.py`, `services/collection_service.py`, and `tests/test_collection.py`, and to confirm the watchlist naming and test patterns before editing. I verified each summary against the source code before making changes, and I used AI again to sanity-check the review-thread context while writing my responses. For Comments 4 and 5, I asked what counterarguments a careful reviewer might raise and used that feedback to make the privacy tradeoff and the case for alphabetical ordering more explicit.

## Comment 1 — Rename
**What I did:** Renamed `save_to_watchlist()` to `add_to_watchlist()` in `services/watchlist_service.py` and updated the only call site in `routes/watchlist/watchlist.py`.
**How I verified:** Ran a project-wide search for `save_to_watchlist` and `add_to_watchlist` with `rg -n "save_to_watchlist|add_to_watchlist" -S .` to confirm there were no missed references.

## Comment 2 — Deduplication
**What I did:** Added `AlreadyInWatchlistError` and a duplicate lookup in `add_to_watchlist()` that checks for an existing `(user_id, film_id)` row before inserting, mirroring `add_to_collection()`. I also taught the watchlist route to return `409 Conflict` for duplicates.
**How I verified:** Compared the new guard to the `existing = CollectionEntry.query.filter_by(...)` pattern in `services/collection_service.py` and confirmed the route now turns that conflict into a JSON error response instead of creating duplicate rows.

## Comment 3 — Missing test
**What I did:** Added `tests/test_watchlist.py` with `test_add_to_watchlist_nonexistent_film_raises`, modeled after `test_add_to_collection_nonexistent_film_raises` in `tests/test_collection.py` and using the same fixture style.
**How I verified:** Used the collection test as the template for the fixture setup, exception assertion, and fake-film-ID pattern, then ran the watchlist test directly with `pytest tests/test_watchlist.py -v`.

## Comment 4 — Default visibility
**My position:** Keep `public=True` as the default for watchlist entries.
**Reasoning:** The watchlist is meant to be a lightweight, shareable “what I want to watch” list, so a public default keeps the common case frictionless and consistent with the app’s social/discoverability direction. Users can still make a different choice later if we add an explicit toggle in the API/UI.
**Tradeoff acknowledged:** A public default is less privacy-friendly than an opt-in model, so the tradeoff is convenience and visibility versus stricter privacy by default.

## Comment 5 — Sort order
**My position:** I would keep alphabetical ordering for the default watchlist response.
**Reasoning:** Alphabetical order makes the list easy to scan when a user is looking for a specific title, and it stays stable instead of reshuffling every time a new film is added. That predictability matters in a list that may grow over time.
**Engagement with reviewer's point:** I agree that date-added order is useful for a “recently added” view, and I understand why that feels more natural for a watchlist. I still prefer alphabetical as the default here, because it optimizes for lookup and keeps the response deterministic; if we later add a second view or sort option, we can support both use cases directly.

## Comment 6 — Rebase
**What conflicted:** The rebase surfaced a `.gitignore` add/add conflict because `main` already ignored `.pytest_cache/`, and once the branch was replayed on top of the UUID refactor, the watchlist service also needed the `WatchlistEntry` model restored against `Film.id` as a UUID.
**How I resolved it:** I merged the `.gitignore` entries, then re-added `WatchlistEntry` to `models.py` with UUID foreign keys so `services/watchlist_service.py` could import it successfully on top of the rebased `main`.
**How I verified no conflict remains:** I checked the rebased file contents directly, confirmed `models.py` defines `WatchlistEntry` again, reran the test suite, and verified `git rev-list --merges origin/main..HEAD` returned no merge commits in the branch-specific history.

## PR Description
This PR adds a watchlist feature so users can save films they want to watch later. It includes a `WatchlistEntry` model, a service layer for adding and listing watchlist items, and REST endpoints for reading and creating watchlist entries.

Design decisions documented in the review responses:
- Watchlist entries default to `public=True` so the common case stays easy and shareable.
- Watchlists remain alphabetical by default so the list is stable and easy to scan.

Manual testing:
1. Start the app with `.venv/bin/python app.py`.
2. Create or reuse a user and a film in the database.
3. Send `POST /watchlist/<user_id>/add` with a valid `film_id` and confirm the API returns `201`.
4. Send the same request again and confirm the API returns `409 Conflict` for the duplicate.
5. Send `POST /watchlist/<user_id>/add` with a nonexistent `film_id` and confirm the API returns `404`.
6. Send `GET /watchlist/<user_id>` and confirm the returned list includes the saved film entries.
