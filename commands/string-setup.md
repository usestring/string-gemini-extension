---
description: Set up String Web Access — check the connection and confirm all three tools respond.
---

Verify that String Web Access is configured and working.

1. Confirm `STRING_API_KEY` is set in the environment. If it is missing, tell the user to
   create a key at https://portal.usestring.ai and export it, then stop.

2. Check the connection with the cheapest possible call — fetch `https://example.com` and
   confirm Markdown comes back.

3. Run one `web_access_search` for a simple query and confirm structured results come back
   with `title`, `url` and `snippet`.

4. Report which tools responded, and what the user should do next if any failed:
   - Empty or error response from both → the key is likely invalid or unset
   - Fetch works but search does not → report the specific error, do not retry in a loop

Keep it to a few lines. This is a connectivity check, not a demo.
