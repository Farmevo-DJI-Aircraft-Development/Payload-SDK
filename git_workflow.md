# Git Workflow

This document outlines the branching strategy and workflow for development on this fork.

## Branch Structure

### `main`
- Synced with the upstream repository (`dji/master`)
- Read-only for most development
- Only receives merges from `dev` when the codebase is stable and ready to contribute back upstream
- Do not commit directly to this branch

### `dev-manifold`
- Primary integration branch for feature development
- Branched from `main`
- All feature branches merge here after development
- Hotfixes merge here after validation
- Tagged when ready for testing

### Feature Branches
- Branched from `dev`
- Naming convention: `feature/short-description` (e.g., `feature/tree-height-measurement`)
- Merged back into `dev` via pull request or merge commit

### Hotfix Branches
- Branched from the commit that a test tag was created on
- Naming convention: `hotfix/issue-description` (e.g., `hotfix/segmentation-edge-case`)
- Merged back into `dev` after fixes are validated

## Workflow

### Starting a New Feature

```bash
git checkout dev-manifold
git pull origin dev-manifold
git checkout -b feature/your-feature-name
```

Develop your feature, commit as needed, then merge into `dev-manifold`.

### Preparing for Testing

1. Ensure `dev-manifold` is in a testable state
2. Create a tag on the current commit in `dev-manifold`:
   ```bash
   git checkout dev-manifold
   git pull origin dev-manifold
   git tag -a v1.0.0-test -m "Testing round 1"
   git push origin v1.0.0-test
   ```
3. Share the tag name with the tester
4. Tester checks out the tag:
   ```bash
   git checkout v1.0.0-test
   ```

### Handling Test Failures (Hotfix)

If bugs are found during testing:

1. Create a hotfix branch from the tagged commit:
   ```bash
   git checkout -b hotfix/issue-description <tag-commit-hash>
   ```
   (Get the commit hash from: `git rev-list -n 1 <tag-name>`)

2. Make your fixes and commit

3. Merge back into `dev-manifold`:
   ```bash
   git checkout dev-manifold
   git pull origin dev-manifold
   git merge --no-ff hotfix/issue-description
   ```

4. Delete the hotfix branch:
   ```bash
   git branch -d hotfix/issue-description
   ```

5. Create a new tag and re-test:
   ```bash
   git tag -a v1.0.0-test-2 -m "Testing round 2"
   git push origin v1.0.0-test-2
   ```

### Syncing with Upstream

Periodically sync `main` with upstream to stay current:

```bash
git checkout main
git fetch dji master
git rebase dji/master
```

If you have local machine-specific changes needed for development, keep them in a separate branch or in gitignored config files—do not commit them to tracked branches.

### Promoting `dev-manifold` to `main`

When `dev-manifold` is stable and ready to merge upstream:

```bash
git checkout main
git pull origin main
git merge --no-ff dev-manifold
git push origin main
```

Then you can sync `main` upstream and potentially contribute the changes back.

## Commit Conventions

All commits must follow these conventions:

### Signing Commits
Always sign your commits using the `-s` flag:
```bash
git commit -s -m "feat: your commit message"
```

This adds a `Signed-off-by` trailer to your commit, indicating authorship and agreement to the changes.

### Commit Message Format
Use conventional commit format with a meaningful type prefix:

```
<type>: <description>

[optional body]
```

**Common types:**
- `feat`: A new feature
- `fix`: A bug fix
- `refactor`: Code refactoring without feature or bug changes
- `docs`: Documentation changes
- `test`: Adding or updating tests
- `perf`: Performance improvements
- `chore`: Build process, dependencies, or tooling changes

**Examples:**
```bash
git commit -s -m "feat: add tree height measurement system"
git commit -s -m "fix: resolve segmentation model edge case"
git commit -s -m "refactor: simplify gimbal control logic"
git commit -s -m "docs: update README with setup instructions"
```

## Key Rules

- **Always sign commits** with `-s`
- **Use conventional commit format** with meaningful type prefixes
- **Never commit directly to `main`**
- **Never commit directly to `dev-manifold`** — use feature branches
- **Always create a feature branch for new work**
- **Use tags, not branches, for marking test points**
- **Use `--no-ff` when merging** to preserve merge history
- **Keep `main` clean** — it should always be in a state ready to sync with upstream

## Glossary

- **Tag**: A named snapshot of a specific commit, used to mark test releases. Lightweight and immutable.
- **Hotfix**: A branch created to fix bugs found during testing, merged back into `dev`.
- **Rebase**: Updating a branch by replaying its commits on top of another branch (used when syncing with upstream).
