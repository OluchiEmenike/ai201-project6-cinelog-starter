# PR Response Doc — CineLog Watchlist Feature

## AI Usage
I asked AI to act as a devil's advocate on my Comment 4 and 5 drafts: “What counterargument would a careful code reviewer raise against this position? What tradeoff am I not acknowledging?” It surfaced two concrete caveats that I added to my reasoning: the watchlist GET route currently returns entries without checking `public`, so the visibility default is metadata rather than access control; and newest-first sorting may not match the order a user intends to watch films.

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
**My position:** Keep watchlist entries public by default.
**Reasoning:** CineLog is a community film-tracking app, and a watchlist can help other users discover films and understand what someone plans to watch. A public default supports that sharing and discovery goal without requiring an extra step. This is a deliberate product choice, not just an inherited database default.
**Tradeoff acknowledged:** Some users treat a watchlist as a personal planning tool and may expect it to be private. A private-by-default policy would better protect that expectation, but would make sharing and discovery opt-in. More importantly, the current `GET /watchlist/<user_id>` route returns entries without checking `public` or authenticating the requester, so the field is currently metadata, not an access-control boundary; the current behavior does not provide a meaningful private alternative. The public default is my product preference, but actual privacy choices require read-side enforcement and an explicit visibility option.

## Comment 5 — Sort order
**My position:** Use date-added order, newest first, as the maintainer suggested.
**Reasoning:** A watchlist is primarily a personal queue of films the user plans to watch. Showing recent additions first makes the newest intentions immediately visible, which is more useful for deciding what to watch next than alphabetical organization.
**Engagement with reviewer's point:** I agree with the maintainer that users commonly want to see what they added recently. Alphabetical order can help locate a title in a long list, but it hides recency and gives new additions no priority. Newest-first date order directly addresses that use case. A counterargument is that the newest addition is not necessarily the next film the user intends to watch; a FIFO or manually ordered queue could better express viewing priority. Given the reviewer's stated preference and the current lack of manual ordering, newest-first is a useful, predictable default. Verified with a test that sets distinct addition dates and asserts the newest film is first.

## Comment 6 — Rebase
**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
The watchlist feature lets users save films they plan to watch and retrieve their list. New entries are marked public by default to support community discovery and sharing; this favors visibility over the privacy a private-by-default policy would offer. At present, the visibility field is returned as metadata and is not enforced by the read route, which returns a user's entries without checking `public`. The add endpoint does not offer a per-entry visibility option.

Manual verification: add films to a watchlist at different times and retrieve the list to confirm the newest addition appears first and each film's `public` value is returned.
