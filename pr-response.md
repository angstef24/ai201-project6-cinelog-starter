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
