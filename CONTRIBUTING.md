# Contributing

Thanks for taking a look. Issues and PRs are welcome. For anything bigger than a small fix, open an issue first so we can agree on the approach before you write code.

## Workflow

1. Branch from `main` (fork first if you don't have write access).
2. Make the change and run the local checks below. They're the same ones CI runs.
3. Open a PR using the template. **The PR title must be a [Conventional Commit](https://www.conventionalcommits.org/)** (`feat:`, `fix:`, `docs:`, `ci:`, `chore:`, `feat!:` for breaking changes). PRs are squash-merged, so the title becomes the commit, and [release-please](https://github.com/googleapis/release-please) uses it to pick the next version and write the changelog.
4. CI must be green and the PR reviewed before merge.

## Local checks

Requires [actionlint](https://github.com/rhysd/actionlint) (bundles shellcheck checks for `run:` blocks when shellcheck is installed).

```bash
actionlint -color
```

To try a workflow change end to end, push your branch and call it from a test repo pinned to the branch's commit SHA. Don't pin to `@main`.

## Workflow conventions

- **Least-privilege `permissions:`** at the job level. Only request `id-token: write` where OIDC is actually used.
- **No long-lived cloud credentials.** Authentication goes through OIDC (`aws-oidc-auth`, `azure-oidc-auth`).
- **Reusable workflows don't call this repo's own composite actions.** A workflow can't pin its own repo to its own SHA, so inline those steps with third-party actions SHA-pinned (see the README's Versioning section).
- **Changes to inputs/outputs are part of the public interface.** Update the README tables, and use `!` in the PR title for anything that breaks existing callers.

## Ground rules

- **No secrets in the repo, ever.** That includes tokens, keys, account/subscription/tenant IDs, and internal hostnames in examples or test fixtures. Use obvious placeholders.
- **Pin third-party GitHub Actions to a full commit SHA** with the version as a trailing comment (`uses: org/action@<sha> # v1.2.3`) so Dependabot can bump them.
- Keep changes focused: one concern per PR makes review and rollback easy.

## Security issues

Don't open a public issue. See [SECURITY.md](SECURITY.md).
