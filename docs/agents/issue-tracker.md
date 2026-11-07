# Issue tracker: GitHub

Issues and specs for this repo live as GitHub issues in [`clementineyyy/BookMind`](https://github.com/clementineyyy/BookMind). Use the `gh` CLI for all operations.

## Conventions

- **Create an issue**: `gh issue create --title "..." --body "..."`.
- **Read an issue**: `gh issue view <number> --json number,title,body,labels,comments`.
- **List issues**: `gh issue list --state open --json number,title,body,labels,comments`.
- **Make an issue a sub-issue of a parent**: `gh issue create --parent <parent> ...`, or `gh issue edit <parent> --add-sub-issue <child>` when supported. Otherwise put `Part of #<parent>` in the child body.
- **Blocking**: use GitHub's native issue dependencies when available; otherwise record `Blocked by: #<n>, #<n>` in the child body. A ticket is unblocked when every blocker is closed.
- **Comment / label / close**: use `gh issue comment`, `gh issue edit --add-label` / `--remove-label`, and `gh issue close`.

Infer the repo from its GitHub remote. File issues only into this repository.

## Pull requests as a triage surface

**PRs as a request surface: no.** `/triage` should process issues, not external PRs, unless this setting is changed.

## When a skill says "publish to the issue tracker"

Create a GitHub issue.

## When a skill says "fetch the relevant ticket"

Read the issue with `gh issue view` as described above.

## Wayfinding operations

Used by `/wayfinder`. The map is a single issue labelled `wayfinder:map`; child decision tickets use `wayfinder:<type>` labels and are linked as sub-issues when supported. Use native issue dependencies for blocking, assign a ticket before working on it, and resolve it with an answer comment followed by closing the issue. Keep the map's Decisions-so-far index updated with a linked gist.
