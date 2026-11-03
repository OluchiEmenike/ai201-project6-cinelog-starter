# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:** Renamed `save_to_watchlist()` to `add_to_watchlist()` in the watchlist service and updated the route import and call site.
**How I verified:** Project-wide Python search found no remaining `save_to_watchlist` references. The service and route imports passed a smoke test, and the existing collection tests passed (4 tests).

## Comment 2 — Deduplication
**What I did:** Followed `add_to_collection()` in `services/collection_service.py`: it checks for an existing entry matching both `user_id` and `film_id`, and raises `AlreadyInCollectionError` instead of creating a duplicate. `add_to_watchlist()` now uses the same lookup pattern and raises `AlreadyOnWatchlistError` when a match exists; the route returns HTTP 409 for that conflict.
**How I verified:** Using an in-memory database, confirmed the first POST returns 201, a repeated POST returns 409, and exactly one watchlist entry is stored. The existing collection test suite also passed (4 tests).

## Comment 3 — Missing test
**What I did:** Added `test_add_to_watchlist_nonexistent_film_raises` in `tests/test_watchlist.py`, using the in-memory app and sample-user fixture pattern from `tests/test_collection.py`.
**How I verified:** Ran `pytest tests/test_watchlist.py -v`; the new test passed.

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
