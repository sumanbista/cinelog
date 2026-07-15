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
**What I did:** Created `tests/test_watchlist.py` with `test_add_to_watchlist_nonexistent_film_raises`, the equivalent of `test_add_to_collection_nonexistent_film_raises` in `tests/test_collection.py`. Reused the same `app` fixture (in-memory SQLite, `db.create_all()`/`db.drop_all()` around the test) and `sample_user` fixture (there's no `conftest.py` yet, so fixtures are duplicated locally the same way the existing test file is self-contained). The test calls `add_to_watchlist()` with a well-formed but nonexistent UUID and asserts it raises `FilmNotFoundError` via `pytest.raises`.
**How I verified:** Ran `pytest tests/test_watchlist.py -v` — 1 passed.

## Comment 4 — Default visibility
**My position:** Keep `public=True` as the default for `WatchlistEntry`. This was an intentional choice, not an inherited default I copy-pasted — I re-examined it specifically because of this comment, and it holds up.

**Reasoning:** CineLog's existing model already makes a visibility decision for the "already watched" side of the product: `CollectionEntry` has no `public` column at all — every logged film is unconditionally visible. There's no opt-out. Given that, a *private-by-default* watchlist would create an odd asymmetry: the films you've committed to and watched are always public, but the films you're merely curious about are hidden unless you actively flip a switch. For a community app, the watchlist is one of the few places a new or lightly-engaged user has activity to show before they've logged anything — it's the thing that makes a profile look alive and gives other users something to react to ("oh you're planning to watch X, have you seen Y?"). If the default were private, the vast majority of users — who never touch settings — would produce empty-looking watchlists on their profiles, which quietly kills the exact discovery/social behavior the feature exists to support. Defaulting to public optimizes for early network effects: visible intent-to-watch is a lightweight, low-commitment social signal that's cheap for users to produce and valuable for the community to see, and it only works if it's on by default.

**Tradeoff acknowledged:** The real cost of public-by-default is that a watchlist can leak more about a user than a diary of watched films does. A watched-films list is retrospective and already vetted by the user (they chose to log it); a watchlist can expose in-progress, unfiltered interest — including films tied to sensitive circumstances (e.g., someone researching a topic privately, or a list that reveals something about their life they're not ready to share) before they've had a chance to curate or reconsider. That's a legitimate reason a more privacy-conservative product would default watchlists to private and treat "make it public" as an explicit, considered action. I'm not dismissing that risk — it's why the field exists as a per-entry, user-controlled boolean rather than being hardcoded to always-public like `CollectionEntry`. Given that escape hatch is in place, I think the discovery upside for a young community-driven app outweighs the exposure risk for the default case, but this is a decision I'd revisit if user feedback showed people were surprised or uncomfortable to find their watchlist entries public.

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
