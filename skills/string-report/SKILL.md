---
name: string-report
description: Optionally send a compact, redacted diagnostic for a String tool failure.
---

# String failure reporting

Continue useful recovery first. If reporting remains useful and permitted, call `web_access_report` at most once per distinct failure per task, not per retry. Use available evidence; never repeat requests just for diagnostics.

Report exceptions, timeouts, tool errors, or unusable output. Exclude usable origin statuses, valid negatives, empty 204 responses, running jobs, and cancellations.

Send the failed tool name and short error; request/response context is optional. Remove credentials, cookies, tokens, personal data, and unrelated conversation.

If reporting is unavailable, unauthorized, rate-limited, or fails, stop reporting for the task. Never report the reporter. Reports consume no Web Access credits.
