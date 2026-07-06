# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename: save_to_watchlist() should follow the project's naming convention. Compare with add_to_collection() — the pattern here is verb_to_noun. Please rename to add_to_watchlist() and update all call sites.
**What I did:** Renamed the service function `save_to_watchlist` to `add_to_watchlist` inside `services/watchlist_service.py` to match CineLog's established `verb_to_noun` naming convention. I also updated the functions and files that called the previous function name.
**How I verified:** Performed a global workspace search for `save_to_watchlist` to guarantee that all references across routes and services were completely updated and no legacy references remained.

## Comment 2 — Deduplication: What happens if a user calls this with a film that's already on their watchlist? The current implementation would add a duplicate entry. Please handle this case.
**What I did:** Added a deduplication check within `add_to_watchlist` by querying `WatchlistEntry` to see if a record already exists matching the given `user_id` and `film_id`. If a match is found, it raises a new custom exception, `AlreadyOnWatchlistError`. I used the same logic found in `services\collection_service.py` for `add_to_collection`
**How I verified:** Added a new test_watchlist.py file with a pytest that verified that adding a duplicate raises an error.

## Comment 3 — Missing test: Please add a test for the case where film_id doesn't exist in the database. Look at the existing tests in test_collection.py — the pattern is there.

**What I did:** 
**How I verified:**

## Comment 4 — Default visibility:
What happens if a user calls this with a film that's already on their watchlist? The current implementation would add a duplicate entry. Please handle this case.

**My position:**
**Reasoning:**
**Tradeoff acknowledged:**

## Comment 5 — Sort order:
I'd prefer watchlists to default to "date added" order rather than alphabetical. Most users want to see what they added recently. I'm open to discussion if you see it differently — but let's make a decision and document it.

**My position:**
**Reasoning:**
**Engagement with reviewer's point:**

## Comment 6 — Rebase:
A refactor merged to main that changed film IDs from integers to UUIDs. Your watchlist code still references integer IDs. Please rebase on main and update accordingly.

**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->