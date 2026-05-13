# Contributing

Thanks for considering a contribution.

## Reporting issues

Open an issue at https://github.com/se4thvin/context-handoff/issues. Useful info to include:

- Claude Code version (`claude --version` if you're on the CLI)
- The exact slash command or phrasing that triggered the behavior
- What you expected vs. what happened
- Relevant excerpts from any handoff file involved

## Pull requests

1. Fork the repo and create a branch off `main`.
2. Keep changes focused — one logical change per PR.
3. If you change a skill's trigger description, add or update the corresponding test in `docs/development/tests/` (TDD-for-skills: baseline subagent behavior captured before and after the change).
4. Run the JSON validators before pushing:
   ```bash
   python3 -m json.tool < .claude-plugin/plugin.json > /dev/null
   python3 -m json.tool < .claude-plugin/marketplace.json > /dev/null
   ```
5. Bump `version` in both `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` if the change is user-visible.
6. Open the PR with a short description of the user-visible change and any breaking notes.

## Release process

1. Make your changes and commit them.
2. Bump version in `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` (use placeholder SHA in marketplace.json initially).
3. Commit, capture the new HEAD SHA, then update `marketplace.json` `source.sha` to that SHA and commit again ("chore: pin marketplace source SHA to vX.Y.Z").
4. Push, tag `vX.Y.Z` on the SHA-pin commit, push the tag.
5. Update `CHANGELOG.md` with the new version's entry.
6. Create a GitHub Release for the tag with release notes.

## Code of conduct

Be decent. Disagreements about technical decisions are welcome; personal attacks are not.
