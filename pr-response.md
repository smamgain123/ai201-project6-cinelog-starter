# PR Response Doc — CineLog Watchlist Feature

## AI Usage

I used ChatGPT to understand the existing codebase patterns, compare the watchlist implementation with the collection service, verify my reasoning for design decisions, and review my commit messages for conventional commit format. I verified all suggested changes against the code and test results before committing.

---

## Comment 1 — Rename

**What I did:**

Renamed `save_to_watchlist()` to `add_to_watchlist()` in `services/watchlist_service.py` and updated the corresponding import and function call in `routes/watchlist/watchlist.py`.

**How I verified:**

I performed a project-wide search to ensure there were no remaining source code references to `save_to_watchlist()`. I then ran the full test suite to verify the rename introduced no regressions.

---
## Comment 2 — Deduplication

**What I did:**

I added a duplicate check to `add_to_watchlist()` before creating a new `WatchlistEntry`. The service queries for an existing entry with the same `user_id` and `film_id`. If one already exists, it raises `AlreadyInWatchlistError` instead of creating a second watchlist entry.

**How I verified:**

I followed the existing implementation in `add_to_collection()` so the watchlist behavior is consistent with the rest of the codebase. I also verified the behavior using a duplicate-entry test to confirm only one watchlist entry is stored for a given user and film.
---

## Comment 3 — Missing Test

**What I did:**

I created a new file, `tests/test_watchlist.py`, and added a test to verify that `add_to_watchlist()` raises `FilmNotFoundError` when a nonexistent `film_id` is provided. This prevents invalid watchlist entries from being created.

**How I verified:**

I modeled the test after the existing `test_add_to_collection_nonexistent_film_raises` in `tests/test_collection.py` so it follows the same fixtures, structure, and assertion style. I ran both the watchlist test and the full test suite to verify the behavior.

---

## Comment 4 — Default Visibility

**My position:**

The watchlist should be private by default.

**Reasoning:**

In CineLog, adding a film to a watchlist is primarily a personal planning action, not necessarily a public recommendation. Users may save films they are only considering, films they are not ready to discuss, or titles they do not want immediately associated with their public profile. A private default reduces hesitation and makes the watchlist feel like a low-friction space for organizing future viewing.

**Tradeoff acknowledged:**

A public default would improve discovery and make it easier for users to browse what others plan to watch. However, that optimizes for social visibility at the cost of user control. I prefer private by default because users can still explicitly make an entry public through the `public` parameter, while avoiding accidental sharing.
---

## Comment 5 — Sort Order

**My position:**

I agree that the watchlist should be sorted by the most recently added films first.

**Reasoning:**

A watchlist is primarily a planning tool rather than a permanent catalog. Users typically add films after discovering them through recommendations, trailers, or friends, and they are most likely to return to the films they added recently. Showing the newest additions first reduces the time needed to find those films and better reflects how people naturally use a watchlist.

**Engagement with the reviewer's point:**

I agree with the maintainer's observation that most users want to see what they added recently. Alphabetical ordering can help when browsing a very large watchlist, but that use case is better addressed with search or an optional sort feature in the future. For the default experience, sorting by date added provides the most useful behavior for the majority of CineLog users.
---

### Comment 6 — Rebase

**What conflicted:**

After rebasing onto the updated `main` branch, the project had migrated film IDs from integers to UUIDs. The watchlist feature still contained the older watchlist model using integer film IDs, so it needed to be updated to match the new schema.

**How I resolved it:**

I restored the `WatchlistEntry` model and updated its `film_id` field from an integer foreign key to a UUID (`db.String(36)`) to match the refactored `Film` model. I also ensured the watchlist model remained consistent with the rest of the codebase.

**How I verified no conflict remains:**

I ran the complete test suite after updating the model and confirmed that all 8 tests passed successfully.
---

## Stretch Feature — Additional Test

**Edge case covered:**

I added a test that verifies adding the same film to a user's watchlist twice raises `AlreadyInWatchlistError` and does not create duplicate database entries.

**Why I selected it:**

This validates the deduplication logic introduced in Comment 2 and ensures the database contains only one watchlist entry for a given user and film combination.
---

## Stretch Feature — Visibility Toggle

**Implementation:**

I updated `add_to_watchlist()` and the watchlist endpoint to accept a `public` parameter.

**Default behavior:**

If no value is provided, the endpoint creates a private watchlist entry by default.

**Caller usage:**

Clients can now include `"public": true` or `"public": false` in the request body when adding a film to explicitly control visibility.
---

## Stretch Feature — Remove From Watchlist

**Implementation:**

I added `remove_from_watchlist(user_id, film_id)` to `services/watchlist_service.py`. The function looks up the matching `WatchlistEntry`, deletes it, commits the transaction, and returns `True`.

**Missing-entry behavior:**

If the film is not currently on the user's watchlist, the function raises `NotInWatchlistError` instead of failing silently. This follows the same pattern used by `remove_from_collection()` and `NotInCollectionError`.

**Test coverage:**

I added one test confirming an existing watchlist entry is removed and another confirming that attempting to remove a missing entry raises `NotInWatchlistError`.

## Commit History Screenshot

![alt text](image.png)

---

# PR Description

## Overview

This PR completes the watchlist feature by allowing users to add films to a watchlist, preventing duplicate entries, supporting removal from the watchlist, and allowing callers to explicitly control watchlist visibility. The implementation follows the existing service and testing patterns already used throughout CineLog.

## Design Decisions

### Default Visibility

Watchlist entries are private by default because adding a film is primarily a personal planning action rather than an intentional public recommendation. Users can explicitly choose to make an entry public through the `public` parameter.

### Sort Order

The watchlist displays the most recently added films first because users are most likely to revisit recently discovered movies rather than browse alphabetically.

## Manual Testing

1. Start the application.
2. Add a valid film to the watchlist.
3. Verify the watchlist entry is created.
4. Attempt to add the same film twice and verify duplicates are prevented.
5. Attempt to add a nonexistent film ID and verify `FilmNotFoundError` is raised.
6. Remove an existing watchlist entry.
7. Attempt to remove a missing watchlist entry and verify `NotInWatchlistError` is raised.
8. Add watchlist entries using both `"public": true` and `"public": false` to verify visibility behavior.
9. Run:

```bash
.venv/bin/python -m pytest tests/ -v
```

Verify all tests pass.