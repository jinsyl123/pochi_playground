# AI Agent Prompt for Pochi X Engagement

**Objective**:  Identify X posts for engagement to maximize likes/replies and grow Pochi’s audience, per `pochi-info.md`. Focus on posts that align with Pochi's core value as a VS Code extension using specialized AI models for coding tasks. 

Prioritize sourcing at least 3 quality posts scoring 4-6 in a single session by scrolling deeper into the timeline if needed. Generate one markdown file per session (e.g., `output/2025-09-04/2025-09-04-1251.md`), organized in a dated folder, containing multiple ENGAGE ✅ posts

**Instructions**:
1. Use Browser MCP to open X like a human. Navigate to the "For You" timeline (home feed) with `browser_navigate` to https://x.com. Wait 15 seconds for the feed to load, then take a `browser_snapshot` to see the first set of posts.
2. Check the posts in the snapshot using `engage-criteria.md` rules. Get details (link, author, content, likes, time, media) for each. Skip bad ones (Step 4) and score the rest (Step 5). If you find 3 or more good posts (4-6 score), jump to Step 7. If not, scroll for more.
3. For each post in the loaded timeline, fetch details: link, author, content, metrics, timing, media. Use `browser_snapshot` to capture visible posts. Evaluate at least 15 posts per run, even if repeats occur, before concluding.
4. Skip Check: Ignore posts if they were engaged with more than 3 times in the last 7 days (check `outputs/` files) or if they’re ads, irrelevant, or too competitive (per `engage-criteria.md`). Log why you skipped and move on. Only score posts that pass.
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

8. After saving the session file (e.g., `output/2025-09-04/2025-09-04-1628.md`), create a PR for that session to https://github.com/TabbyML/snowshoe using `create-pr.md`. Use a unique branch like `docs/session-analysis-2025-09-04-1628`, a commit message `docs(session): add 2025-09-04 16:28 analysis`, and title `Session analysis - 2025-09-04 16:28`. Add Lucy (username: gyxlucy) as reviewer with `--reviewer gyxlucy`. Work in the `snowshoe` folder.

Only add and commit the new session file to the branch from the local snowshoe repo (e.g., `git add output/2025-09-04/2025-09-04-1628.md` and `git commit -m "docs(session): add 2025-09-04 16:28 analysis"`). Check GitHub login with `gh auth status`; if not logged in, run `gh auth login` with a token for `TabbyML/snowshoe` (needs `repo` access). Skip the PR if fewer than 3 good posts (log: 'Fewer than 3 posts; no PR'). If the PR fails, log the issue, push the branch, and give a manual link (e.g., https://github.com/TabbyML/snowshoe/pull/new/docs/session-analysis-YYYY-MM-DD-HHMM). Print: 'Generated session file output/2025-09-04/2025-09-04-1628.md with X good posts. PR made if X >= 3.'