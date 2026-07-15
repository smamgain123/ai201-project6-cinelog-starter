---

# PR Description

## Overview

This PR completes the watchlist feature by allowing users to add films to their watchlist, preventing duplicate entries, supporting removal from the watchlist, and allowing callers to explicitly control watchlist visibility. The implementation follows the existing service and testing patterns already used in the CineLog codebase.

## Design Decisions

### Default Visibility

Watchlist entries are private by default because adding a film is primarily a personal planning action rather than an intentional public recommendation. Users can explicitly choose to make an entry public by passing the `public` parameter.

### Sort Order

The watchlist is intended to display the most recently added films first. Since users generally revisit recently discovered films, this ordering better matches common watchlist behavior than alphabetical ordering.

## Manual Testing

1. Start the application.
2. Add a valid film to the watchlist.
3. Verify the watchlist entry is created.
4. Attempt to add the same film twice and verify an error is raised.
5. Attempt to add a nonexistent film ID and verify `FilmNotFoundError` is raised.
6. Remove an existing watchlist entry and verify it is deleted.
7. Attempt to remove a film that is not on the watchlist and verify `NotInWatchlistError` is raised.
8. Add watchlist entries using both `"public": true` and `"public": false` and verify visibility is stored correctly.
9. Run:

```bash
.venv/bin/python -m pytest tests/ -v
```

Verify all tests pass successfully.
