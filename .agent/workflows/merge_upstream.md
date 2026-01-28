---
description: Sync repository with upstream Invidious, preserving local customizations (UI, Config, patches).
---

# Merge Upstream with Custom Preservation

This workflow guides the AI to merge changes from the official Invidious repository while strictly protecting our local customizations (Modern UI, 1panel Config, Refresh Job Fixes).

### 1. Prepare Environment
1. Check if `upstream` remote exists. If not, add it:
   `git remote add upstream https://github.com/iv-org/invidious.git`
2. Fetch the latest changes:
   `git fetch upstream`

### 2. Attempt Merge
// turbo
1. Try to merge the upstream master branch:
   `git merge upstream/master`

### 3. Handle Conflicts (Conditional)
If the merge step above returns "Already up to date", stop.
If the merge is successful (no conflicts), stop.
**If there are CONFLICTS**, perform the following for each conflicting file:

1. **Identify the File Type**:
   - **Config Files** (`config/config.yml` or similar): **STRONGLY PREFER** our local version. Our 1panel paths and Postgres tweaks are critical.
   - **CSS/Assets** (`assets/css/*`): **STRONGLY PREFER** our local version (Modern UI). 
     - *Action*: Keep our local CSS block. If upstream added *new* selectors, try to append them, but do not let them overwrite our `modern_ui.css` imports or overrides.
   - **Patched Logic** (`src/invidious/jobs/refresh_feeds_job.cr`, `src/invidious/trending.cr`):
     - View the conflict markers `<<<<<<<`, `=======`, `>>>>>>>`.
     - **CRITICAL**: We *must* keep our "DB Connection Starvation Fix" in `refresh_feeds_job.cr` (loading emails into array first).
     - **CRITICAL**: We *must* keep our "Trending 400 Error Fix" in `trending.cr`.
     - For other logic changes, generally accept Upstream's new code.
   - **Unknown Logic**: Default to investigating. If it looks like a general feature update, accept Upstream.

2. **Resolution Process**:
   - Use `view_file` to read the file with conflict markers.
   - Use `replace_file_content` to essentially write the "Resolved" version of the file (removing markers).
   - `git add <file>` after resolution.

### 4. Finalize
1. Once all conflicts are resolved:
   `git commit -m "Merge upstream changes (Preserving Local/1Panel optimizations)"`
2. Push the changes:
   `git push`
