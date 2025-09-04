# AI Agent Prompt for Pochi X Engagement

**Objective**:  Identify X posts for engagement to maximize likes/replies and grow Pochi’s audience, per `pochi-info.md`. Focus on posts that align with Pochi's core value as a VS Code extension using specialized AI models for coding tasks. Prioritize sourcing at least 3 quality posts scoring 4-6 each execution by scrolling deeper into the timeline if needed.

**Instructions**:
1. Use Browser MCP to open X like a human. Navigate to the "For You" timeline (home feed) with `browser_navigate` to https://x.com.
2. Scroll naturally using `browser_press_key` with "PageDown". Pause 10-15 seconds between major actions. Wait 10-15 seconds after scrolling. If fewer than 3 qualifying posts are found in the initial load, automatically scroll further (repeat "PageDown" 8-10 times per batch, up to 3 batches) to load more content. Verify new posts are loaded by comparing post IDs or content in `browser_snapshot`. Stop if CAPTCHA appears and log: "CAPTCHA detected, halting scroll."
3. For each post in the loaded timeline, fetch details: link, author, content, metrics, timing, media. Use `browser_snapshot` to capture visible posts. Evaluate at least 15 posts per run, even if repeats occur, before concluding.
4. Skip Check: Review `outputs/YYYY-MM-DD.md` (last 7 days) for prior engagements (>3/week, skip). For each post, IMMEDIATELY check skip conditions from `engage-criteria.md`. IF ANY SKIP CONDITION = TRUE: Log as skipped, DO NOT SCORE, move to next post.ONLY if ALL skip checks = FALSE, proceed to scoring.
5. For non-skipped posts, score and generate reply per `engagement-criteria.md`. Perform quick sanity check: Is reply on-topic, accurate to Pochi (per pochi-info.md), and human-like? If No, SKIP or revise. Mandatory log per post: “Score: [X/6]. Sanity Check: [Pass/Fail, Reason (e.g., 'Accurate to Pochi’s VS Code focus, human-like tone')].”
6. Continue scrolling, loading, and evaluating posts until at least 3 posts scoring 4-6 are identified or 50 posts are evaluated. Log scroll attempts and new posts loaded per batch.
7. Save results to `outputs/YYYY-MM-DD.md`. If the file exists, append at the end of document with a new section header `## YYYY-MM-DD HH:MM` (use current timestamp). Structure the output under `## Detailed Analysis`, listing posts as `### POST N - ENGAGE ✅` for engages or `### POST N - SKIP ❌` for skips. Include a separate `### Skipped Posts` section with brief logs for all skipped posts (e.g., Link, Author, Skip Reason). End with a `### Summary`: Posts analyzed: X, Posts scoring 4-6: Y (with scores), Recommendations: [e.g., Prioritize high-engagement topics like specialized models].
8. After saving, print: “Found X posts scoring 4-6: [scores]. Saved to `outputs/YYYY-MM-DD.md.`”

Use this per-post template:
- **Link**: [link]
- **Author**: [@username]
- **Post Content** (Excerpt): [Concise excerpt of the post text, including key quotes or phrases. Describe any media, e.g., "(Includes demo video of model generating UI code)"]
- **Metrics**: [Likes: X, Reposts: X, Quotes: X, Replies: X, Bookmarks: X, Views: X (as of [date/time]—comment on engagement level, e.g., 'Strong for niche AI topic')"]
- **Skip Check**: [Yes/No, Reason, including prior engagements count]
- **Analysis**: Score [X/6] (Engagement: [X/2], Relevance: [X/3], Opportunity: [X/1])
- **Suggested Reply**: [reply]
- **Sanity Check**: [Pass/Fail, Reason]
- **Reply Strengths**: [Explanation covering alignment, tone, engagement]
- **Edits/Feedback**: [Lucy: Add comments here, e.g., “Posted as-is” or “Edited: [Changes], Reason: [Why]” or “Skipped: [Reason]”] Include skipped posts section and summary: posts analyzed, posts scoring 4-6, scores, recommendations.