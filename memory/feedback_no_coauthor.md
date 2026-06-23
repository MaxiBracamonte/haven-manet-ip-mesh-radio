---
name: No co-author lines in commits
description: Do not add Co-Authored-By Claude trailer to git commits
type: feedback
---

Do not add `Co-Authored-By:` lines to git commit messages.

**Why:** User does not want Claude attribution appearing on GitHub commits.

**How to apply:** Every time creating a git commit, omit the `Co-Authored-By: Claude ...` trailer entirely.
