# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:** Renamed `save_to_watchlist()` to `add_to_watchlist()` in `services/watchlist_service.py` to match the project's `verb_to_noun` naming convention used elsewhere (e.g. `add_to_collection()`, `remove_from_collection()`). Updated the one call site in `routes/watchlist/watchlist.py` (both the import statement and the function call).
**How I verified:** Ran `grep -rn "save_to_watchlist" --include="*.py" .` across the repo before and after the change — found exactly 3 references (definition + import + call site) beforehand, 0 remaining afterward.

## Comment 2 — Deduplication
**What I did:** Added a duplicate check to `add_to_watchlist()` in `services/watchlist_service.py`, mirroring the pattern in `add_to_collection()` (`services/collection_service.py`): after confirming the film exists, query for an existing `WatchlistEntry` matching `(user_id, film_id)`. If one is found, raise a new `AlreadyInWatchlistError` (defined in `watchlist_service.py`, alongside the existing `FilmNotFoundError` reuse) instead of creating a second entry. Only if no existing entry is found does the function create and commit the new `WatchlistEntry`.
**How I verified:** Read `add_to_collection()` first to confirm the existing convention — it does a `.filter_by(user_id=..., film_id=...).first()` lookup and raises a dedicated `AlreadyInCollectionError` before creating the entry — and followed the same shape rather than inventing a different check (e.g. relying solely on a DB unique constraint / IntegrityError). Manually traced the new code path: existing entry found → `AlreadyInWatchlistError` raised, no new row created; no existing entry → entry created and committed as before.

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
