# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:** I changed the name of the save_to_watchlist function to add_to_watchlist. I then went and found all places where this function was called to make sure they all called the correctly named function.
**How I verified:** I ran grep -rn "save_to_watchlist" --include="*.py" in the terminal to be sure the old name wasn't found in any python files.

## Comment 2 — Deduplication
**What I did:** I added dedplication code to the add_to_watchlist function.
**How I verified:** I compared the code to that of the add_to_collection function as well. I needed to add a UniqueConstraint to the models.py file as well to confirm no duplicate will slip.

## Comment 3 — Missing test
**What I did:** I created a new file test_watchlist.py and used the same pattern as test_add_to_collection_nonexistent_film_raises to create test_add_to_watchlist_nonexistent_film_raises.
**How I verified:** I ran pytest tests/test_watchlist.py -v to make sure that the test case passed. At first, it did not since I needed to add in all of the imports.

## Comment 4 — Default visibility
**My position:** We should change the default from public=True to public=False on the watch lists.
**Reasoning:** We need to make the platform safe and enact privacy for those who may not know or don't act fast enough to change their profile to private. Even making it clear to the user that the account starts as public relies on their action to do the work rather.
**Tradeoff acknowledged:** The goal of our platform is to promote social interactions across people who are avid or casual movie watchers alike. By having your watch list be public, users may able to find others with similar interests outside of their usual circle and thus creating an overall tighter community. A user starting with a private account and not switching to public right away as they want would mean they miss out on certain social feature for a little bit. However, the negative impact of having an account be public without realizing can already provide others with the ability to scrape and see a user's personal watch list data without realizing. I think a further discussio could be had about including a prompt in the beginning of account creation for the user to be able to toggle themselves rather than sticking with a default. Either way, I stand firm in my belief the current code should be public=False.


## Comment 5 — Sort order
**My position:** I agree that the sort should be on date added.
**Engagement with reviewer's point:** I agree with the point that most users want to see what they have added recently. I would argue even further that users are more likely to be able to remeber a general timeframe when they watched a movie over what the title may have been. I also feel that title introduces another element where we would need to decide if titles with "The" should be with the "T"'s or if we start with the first letter of the second word. Date added is much easier to grasp.

## Comment 6 — Rebase
**What conflicted:** Running `git rebase origin/main` conflicted in two files. `.gitignore` had overlapping entries added on both sides. `models.py` conflicted because main had migrated `Film.id` from an integer to a UUID (`db.String(36)`) before my watchlist commits were written, so my code still assumed integer film IDs.
**How I resolved it:** For `.gitignore`, I kept the union of both sides' entries. For `models.py`, I updated `Film.id`, `CollectionEntry.film_id`, and `WatchlistEntry.film_id` to all use `db.String(36)` so the watchlist code matches main's UUID schema, then continued the rebase commit by commit with `git add` and `git rebase --continue`.
**How I verified no conflict remains:** I ran the full test suite (`pytest`) to confirm everything still passes with UUID-typed IDs, and ran `git log --oneline --graph` to confirm the branch history is a straight line with no new merge commits.

## PR Description
### Overview
Adds a watchlist feature to CineLog, letting users save films they want to watch, separate from their collection of already-watched films (`CollectionEntry`). Introduces a new `WatchlistEntry` model and two endpoints:
- `GET /watchlist/<user_id>` — list a user's watchlist, sorted by most recently added first.
- `POST /watchlist/<user_id>/add` — add a film (`{"film_id": <uuid>}`) to a user's watchlist.

### Design decisions
- `WatchlistEntry` mirrors the shape of `CollectionEntry` (UUID primary key, `user_id`/`film_id` foreign keys, `date_added` timestamp), plus a `public` boolean for social-sharing.
- Watchlists default to `public=False`. Since most users never change a default, defaulting to public would expose personal watchlist data to scraping/viewing before a user thinks to lock it down — an asymmetric and hard-to-reverse risk compared to a user opting in to sharing later.
- `get_watchlist` sorts by `date_added` descending (most recent first), matching `get_collection`'s existing convention and prioritizing what users are most likely to want to see first.
- A `(user_id, film_id)` unique constraint (`unique_user_film_watchlist`) prevents duplicate watchlist entries at the database level, backing up the application-level check in `add_to_watchlist`.
- `add_to_watchlist` raises `FilmNotFoundError` for a nonexistent film and `AlreadyInWatchlistError` for a duplicate, matching the existing `collection_service` error-handling pattern.
- Rebased onto `main` to pick up the integer → UUID migration for `Film.id`; all watchlist code and the `WatchlistEntry.film_id` column use `db.String(36)` to match.

### Manual testing steps
1. Start the app and create a user and a film (or seed data).
2. `POST /watchlist/<user_id>/add` with a valid `film_id` — confirm a `201` response with the new entry, and that `public` is `false` by default.
3. Repeat the same request — confirm it's rejected as a duplicate rather than creating a second row.
4. `POST` with a made-up `film_id` — confirm a not-found error rather than a raw database error.
5. Add a second film, then `GET /watchlist/<user_id>` — confirm the most recently added film appears first.
6. Run `pytest` — all tests pass.
