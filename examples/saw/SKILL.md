---
name: saw
description: "Parallel agent coordination: Scout analyzes the codebase and produces a plan; Wave agents implement in parallel."
triggers:
  - match: "^/saw program"
    inject: references/program-flow.md
  - match: "^/saw amend"
    inject: references/amend-flow.md
  - match: "failure|blocked|partial|E19|E25|E26"
    inject: references/failure-routing.md
---

# Scout-and-Wave: Parallel Agent Coordination

This is an example frontmatter showing how SAW declares triggers for context injection.

The `triggers:` block tells `scripts/inject-context` which reference files to load based on the user's prompt. For example:

- `/saw program execute "add caching"` matches `^/saw program` and injects `references/program-flow.md`
- `/saw amend --add-wave` matches `^/saw amend` and injects `references/amend-flow.md`
- "agent B failed with timeout" matches `failure|blocked|partial` and injects `references/failure-routing.md`

Prompts that don't match any trigger (e.g., `/saw wave`, `/saw scout`) inject nothing -- zero overhead.
