# Maintaining This Skill

This directory is the `tanstack-query-best-practices` Skill: `SKILL.md` is the entry point and `references/` holds topic guidance.

- Keep triggering conditions in the `SKILL.md` frontmatter `description`, and keep technical rules and examples in `references/`.
- When adding a reference, list it in `SKILL.md` with the topics it covers.
- Keep each rule in one reference; link to it instead of repeating it elsewhere.
- Examples share one domain (`usersQueryOptions`, `UsersResult` with `items`, `totalCount`, `activeCount`), use `function` declarations and method shorthand, and target TanStack Query v5.
- Keep guidance scoped to the code being written or edited; it is not a mandate to refactor unrelated consumers.
- For instruction-only edits, check that reference paths, names, and examples stay consistent across files.
