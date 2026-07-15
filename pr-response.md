# PR Response Doc — CineLog Watchlist Feature

## AI Usage
I used AI to help me understand the initial codebase and to act as a sounding board for my decisions.  I asked it to create the counterpoint to ensure my arguments were sound.  It also helped me when I had a merging issue when my rebase command did not work correctly, helping me debug. 


## Comment 1 — Rename
**What I did:** Used global find-and-replace to update the function definition and its call site in the route
**How I verified:** Used git diff to ensure all instances were changed

## Comment 2 — Deduplication
**What I did:** Looked at the logic in the add_to_collections and created the code for add_to_watchlist using the same logic.
**How I verified:** Ran the pytest and confirmed it passed all tests written.

## Comment 3 — Missing test
**What I did:** Created tests/test_watchlist.py and added test_add_to_watchlist_nonexistent_film_raises based on the collection test patterns.
**How I verified:** Ran the pytest and confirmed it passed all the tests written.

## Comment 4 — Default visibility
**My position:** Keep public=True default
**Reasoning:** Most social media apps allow for user content to be public by default to encourage sharing with friends.  This will create a larger community base rather than having private as default and users needing to navigate settings to make it public. 
**Tradeoff acknowledged:** The tradeoff would involve privacy, as users may not realize their movie lists are public by default and may not want to share what they are watching with the world.  But they can easily switch the setting off to private instead if needed, so their privacy is taken into account. 

## Comment 5 — Sort order
**My position:** I agree with making the sort order by date added instead of alphabetical.
**Reasoning:** A watchlist functions as a dynamic priority queue for what a user intends to watch next. When a user adds a film, their intent to watch it is usually at the top. Sorting alphabetically acts more like a static library index, which buries new additions if their titles happen to fall later in the alphabet.
**Engagement with reviewer's point:** I agree that most users will want to see things they added more recently.  Most users will want to view their most added entries to a watch list. 

## Comment 6 — Rebase
**What conflicted:** The rebase completed but it created two silent merge conflicts.  The git-auto merge algorithm dropped the watchEntry class from Models.py and the original watchlist code still expected film_id ot be an integer unlike the UUID standard.
**How I resolved it:** I manually restored the missing WatchlistEntry class at the bottom of models.py and updated its film_id column to use db.String(36) instead of db.Integer. I also updated the add_to_watchlist docstring in watchlist_service.py to reflect that film_id is now a UUID string.
**How I verified no conflict remains:** I verified the models.py file contains the restored class with the correct String datatypes, and pytest tests/ confirms the app still boots successfully with the new schema.

## PR Description
## Feature Overview
This PR adds a comprehensive Watchlist feature to CineLog, allowing users to save films they intend to watch. The implementation includes the new `WatchlistEntry` database model, service-layer business logic (with deduplication), and REST API endpoints for adding and retrieving watchlist entries.

## Design Decisions
* **Visibility Default:** I chose to maintain `public=True` as the default setting for watchlists. CineLog is fundamentally a social, community-driven platform. Public watchlists minimize friction for discovery and sharing among friends. While this does trade off strict privacy by default, users can still explicitly toggle this setting if they choose. 
* **Sort Order:** I updated the `get_watchlist` return order to sort by `date_added` (descending) rather than alphabetically. A watchlist functions as a dynamic priority queue; users typically want to see their freshest, highest-intent additions at the top when deciding what to watch next. 

## Manual Testing Steps
1. Checkout this branch and ensure your virtual environment is active.
2. Run the automated test suite to confirm the new `WatchlistEntry` logic and deduplication work as expected: `pytest tests/ -v`
3. Start the local server: `python app.py`
4. Use an API client (like Postman or cURL) to send a `POST` request to `/watchlist/<user_id>/add` with a valid JSON body containing a `film_id`.
5. Send a `GET` request to `/watchlist/<user_id>` to verify the new film appears at the top of the list with the correct metadata.


## Screenshot

<img width="729" height="184" alt="Commit_List" src="https://github.com/user-attachments/assets/d12175c1-20e1-40e9-9cbf-351f590eddaf" />
