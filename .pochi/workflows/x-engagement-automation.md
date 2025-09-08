# AI Agent Prompt for Pochi X Engagement

**Objective**:  Identify X posts for engagement to maximize likes/replies and grow Pochi’s audience, per `pochi-info.md`. Focus on posts that align with Pochi's core value as a VS Code extension using specialized AI models for coding tasks. 

Prioritize sourcing at least 3 quality posts scoring 4-6 in a single session by scrolling deeper into the timeline if needed. Generate one markdown file per session (e.g., `output/2025-09-04/2025-09-04-1251.md`), organized in a dated folder, containing multiple ENGAGE ✅ posts

**Instructions**:
1. Initialize a persistent seen_posts.json file (if not exists) to track all posts seen across sessions. Run cleanup to remove entries older than 7 days:
``` bash
[ -f seen_posts.json ] || echo "{}" > seen_posts.json
jq 'to_entries | map(select(.value.timestamp > (now - 7*24*60*60 | strftime("%Y-%m-%d %H:%M:%S")))) | from_entries' seen_posts.json > tmp.json && mv tmp.json seen_posts.json
```
2. Use Browser MCP to open X like a human. Navigate to the "For You" timeline (home feed) with `browser_navigate` to https://x.com. Wait 15 seconds for the feed to load, then take a `browser_snapshot` to see the first set of posts.
2. Check the posts in the snapshot using `engage-criteria.md` rules. Get details (link, author, content, likes, time, media) for each. For each post, check seen_posts.json to skip duplicates from any session. Log "Skipped: Post [link] seen on [timestamp]" if found. Skip bad ones (Step 4) and score the rest (Step 5). If you find 3 or more good posts (4-6 score), jump to Step 7. If not, scroll for more.
3. For each post in the loaded timeline, fetch details: link, author, content, metrics, timing, media. Use `browser_snapshot` to capture visible posts. Add new post IDs/links to seen_posts.json with timestamp:
``` bash
TIMESTAMP=$(date -u +"%Y-%m-%d %H:%M:%S")
jq --arg link "$link" --arg time "$TIMESTAMP" '.[$link] = {timestamp: $time}' seen_posts.json > tmp.json && mv tmp.json seen_posts.json
```
Evaluate at least 15 posts per run, even if repeats occur, before concluding.
4. Skip Check: Ignore posts if:
- They appear in `seen_posts.json` (logged in Step 2).
- They were engaged with more than 3 times in the last 7 days (check outputs/ files and PR requests in the snowshoe repo).
- They’re ads, irrelevant, or too competitive (per `engage-criteria.md`). Log why you skipped (e.g., "Skipped: Post [link] seen on [timestamp]" or "Engaged 4 times") and move on. Only score posts that pass.
5. Scoring: Rate good posts 0-6 based on `engage-criteria.md` (engagement: 2, relevance: 3, opportunity: 1). A 4-6 score is "ENGAGE ✅". Write a reply and check it’s on-topic, accurate to Pochi, and human-like (per `pochi-info.md`). If not, skip or fix it. Log the score and check result (e.g., “Score: 5/6. Sanity Check: Pass”).
6. If fewer than 3 good posts are found, scroll to load more. Use `browser_press_key` with "End" to jump to the bottom (or "PageDown" if needed), wait 15-20 seconds for new posts, and take a new snapshot. Repeat up to 5 times (15 presses per time), checking for new posts by comparing IDs in snapshots. Log “New posts: [number]” or “Feed stagnant” if none. If stagnant after 2 tries, refresh with `browser_navigate` to https://x.com/home`, wait 15 seconds, and try again. Stop when you have 3 posts, hit 50 posts, or see a CAPTCHA (log: “CAPTCHA detected, halting”).
7. Save all good posts (4-6 score) in one file named by date and time (e.g., `output/2025-09-04/2025-09-04-1607.md`). Include:
- A header `## Session Analysis - [YYYY-MM-DD HH:MM]`.
- **Summary**: Number of posts analyzed, number of ENGAGE ✅ posts (target: 3+).
- `### ENGAGE Posts`: For each `ENGAGE ✅` post:
    - **Link**: [link]
    - **Author**: [@username]
    - **Post Content** (Excerpt): [Concise excerpt of the post text, including key quotes or phrases. Describe any media, e.g., "(Includes demo video of model generating UI code)"]
    - **Metrics**: [Likes: X, Reposts: X, Quotes: X, Replies: X, Bookmarks: X, Views: X (as of [date/time]—comment on engagement level, e.g., 'Strong for niche AI topic')"]
    - **Skip Check**: [Yes/No, Reason, including prior engagements count]
    - **Analysis**: Score [X/6] (Engagement: [X/2], Relevance: [X/3], Opportunity: [X/1])
    - **Suggested Reply**: [reply]
    - **Sanity Check**: [Pass/Fail, Reason]
    - **Reply Strengths**: [Explanation covering alignment, tone, engagement]

**Step 8**: Commit only this new session file
``` bash
git reset  # Clear Git index
git checkout main
git pull origin main
BRANCH="docs/session-analysis-$TIMESTAMP"
git checkout -b "$BRANCH"
git reset
git status --short
git add "$OUTPUT_FILE"
COMMIT_MSG="docs(session): add $(echo $TIMESTAMP | sed 's/- / /') analysis"
git commit -m "$COMMIT_MSG" || { echo "Commit failed"; exit 1; }
```
**Step 9**: Create pull request
``` bash
git remote get-url origin || git remote add origin https://github.com/TabbyML/snowshoe.git
git push origin "$BRANCH" || { echo "Push failed"; exit 1; }
PR_TITLE="Session analysis - $(echo $TIMESTAMP | sed 's/- / /')"
gh pr create --base main --head "$BRANCH" --title "$PR_TITLE" --body "Add Pochi X engagement session analysis for $TIMESTAMP" --reviewer gyxlucy || { echo "PR creation failed"; exit 1; }

# Restore stashed changes
if git stash list | grep -q "Auto-stash"; then
  git stash pop
fi
```

**Step 10**: Update check_skip.sh to handle cross-session deduplication
``` bash
cat << EOF > scripts/check_skip.sh
#!/bin/bash
post="\$1"
link=\$(echo "\$post" | jq -r '.link')
[ -f engagement_cache.json ] || echo "{}" > engagement_cache.json
ENGAGE_COUNT=\$(jq --arg link "\$link" '.[ \$link] // 0' engagement_cache.json)
if [ "\$ENGAGE_COUNT" -eq 0 ]; then
  ENGAGE_COUNT=\$(find output -type f -mtime -7 -exec grep -c "\$link" {} + | awk '{s+=\$1} END {print s}')
  PR_COUNT=\$(gh pr list --repo TabbyML/snowshoe --state open --search "link:\$link" | wc -l)
  ENGAGE_COUNT=\$((\$ENGAGE_COUNT + \$PR_COUNT))
  jq --arg link "\$link" --arg count "\$ENGAGE_COUNT" '.[ \$link] = (\$count | tonumber)' engagement_cache.json > tmp.json && mv tmp.json engagement_cache.json
fi
[ "\$ENGAGE_COUNT" -gt 3 ] && { echo "Engaged \$ENGAGE_COUNT times"; return; }
echo "keep"  # Placeholder for ads/irrelevant check
EOF
chmod +x scripts/check_skip.sh
```

**Step 11**: Update `config.sh` to repeat rate threshold
cat << EOF > config.sh
MAX_ATTEMPTS=5
MAX_POSTS=50
EOF
```

