# Available Skills

## `/saw` -- Scout-and-Wave Parallel Agent Coordination

Use `/saw` for any feature work that can be decomposed across files and run in
parallel. Scout analyzes the codebase and produces a coordination plan (IMPL
doc); Wave agents implement their assigned files simultaneously.

**When to reach for it:** adding a feature, refactoring across multiple files,
porting a design, or bootstrapping a new project from scratch.

**Subcommands:**

| Command | Purpose |
|---------|---------|
| `/saw scout "<feature>"` | Analyze codebase, produce IMPL doc |
| `/saw wave` | Execute next pending wave |
| `/saw wave --auto` | Execute all remaining waves unattended |
| `/saw status` | Show current wave and agent progress |
| `/saw bootstrap "<project>"` | Design new project structure from scratch |
| `/saw interview "<description>"` | Structured requirements gathering |
| `/saw program plan/execute/status/replan` | Multi-IMPL program coordination |
| `/saw amend --add-wave/--redirect-agent/--extend-scope` | Modify active IMPL |

**Typical flow:**
```
/saw scout "add a cache to the API"   # Scout analyzes (~30-90s)
/saw wave                              # Agents execute in parallel (~2-5min)
```
