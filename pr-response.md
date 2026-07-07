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
**What I did:** Added a new pytest called `test_add_to_watchlist_nonexistent_film_raises` to `tests/test_watchlist.py`. This test specifically asserts that invoking `add_to_watchlist` with an invalid or missing `film_id` correctly raises a domain-level `FilmNotFoundError`.
**How I verified:** Mirrored the logic assertion patterns used in `test_add_to_collection_nonexistent_film_raises` from `tests/test_collection.py`. Verified the implementation by executing `pytest tests/test_watchlist.py -v` in the terminal, confirming the test suite executes and passes cleanly.

## Comment 4 — Default visibility: I notice watchlists default to public=True. We don't have a documented decision on default visibility for user lists. Before I can approve this, I need you to add a note to your PR description explaining your reasoning. I want to make sure we're being intentional here, not just inheriting a default.

**My position:** I have kept the default visibility as public=True.
**Reasoning:** CineLog is built to be community-centric and encourages users sharing their lists and interactions with others. Because our goal is to foster community, we will leave turning off visibility as opt-in rather than the default.
**Tradeoff acknowledged:** The primary tradeoff of a public default is user privacy. Users who want to curate a private or experimental watchlist might be caught off guard if they assume their list is confidential by default. However, this is heavily mitigated by giving users an explicit parameter to opt out and implementing a reminder to the user that the default visibility is public balancing social platform optimization with user privacy.

## Comment 5 — Sort order:
I'd prefer watchlists to default to "date added" order rather than alphabetical. Most users want to see what they added recently. I'm open to discussion if you see it differently — but let's make a decision and document it.

**My position:** I agree that watchlists should be shorted by "date added" instead of alphabetical and have left it. I have changed ordering by `Film.title.asc()` to `WatchlistEntry.date_added.desc()` from alphabetical to descending by date added.
**Reasoning:** Most users want to view their own watchlists and think of movies that they were most recently thinking of, not thinking of them alphabetically.
**Engagement with reviewer's point:** I agree with the reviewer that most users want to see their watchlists by what they most recently added. Alphabetical sorting is more useful for static, categorical collections, while a watchlist is more dynamic.

## Comment 6 — Rebase:
A refactor merged to main that changed film IDs from integers to UUIDs. Your watchlist code still references integer IDs. Please rebase on main and update accordingly.

**What conflicted:** Merging the latest updates from the `main` branch created a structural conflict because a recent refactor migrated all core film identifiers (`film_id`) from sequential integers to globally unique identifiers (UUIDs). This broke my initial implementation of `add_to_watchlist`, which explicitly expected and handled integer types for `film_id`, causing structural type mismatches and test failures with the updated database schema.
**How I resolved it:** I executed a git rebase onto `main` (`git fetch origin` followed by `git rebase origin/main`). During the interactive conflict resolution, I updated the model mappings and docstrings within `services/watchlist_service.py` to seamlessly handle string-based UUID values. I also went into my newly created test file, `tests/test_watchlist.py`, and swapped out the hardcoded mock integer values (like `99999`) for a valid, standard UUID string format (e.g., `"00000000-0000-0000-0000-000000000000"`).
**How I verified no conflict remains:** Ran a clean git status check to verify the rebase successfully concluded with zero unresolved merge markers. I then executed the entire verification test suite locally using `pytest tests/ -v` to ensure the new UUID-driven watchlist queries process correctly and no database integrity errors are thrown.
## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->