---
name: git-commit
description: Stage all changes and create a Conventional Commits git commit with a clear title and description explaining what changed and why.
disable-model-invocation: true
argument-hint: "[add-scope: to include scope] [no-description: to skip commit message description]"
---

Create a git commit following Conventional Commits. Follow each step precisely.

## Step 1 — Gather context

Run both commands in parallel:

1. `git status --short`
2. `git diff HEAD -- . \
':(exclude)*.lock' \
':(exclude)yarn.lock' \
':(exclude)package-lock.json' \
':(exclude)pnpm-lock.yaml' \
':(exclude)composer.lock' \
':(exclude)Podfile.lock' \
':(exclude)Gemfile.lock' \
':(exclude)pubspec.lock' \
':(exclude)*.min.js' \
':(exclude)*.min.css' \
':(exclude)*.map' \
':(exclude)*.snap' \
':(exclude)dist/**' \
':(exclude)build/**' \
':(exclude).next/**' \
':(exclude).nuxt/**' \
':(exclude).expo/**' \
':(exclude).dart_tool/**' \
':(exclude)android/build/**' \
':(exclude)android/.gradle/**' \
':(exclude)ios/build/**' \
':(exclude)ios/Pods/**' \
':(exclude).gradle/**' \
':(exclude)node_modules/**' \
':(exclude)vendor/**' \
':(exclude)__pycache__/**' \
':(exclude)*.generated.*' \
':(exclude)*.g.dart' \
':(exclude)*.freezed.dart' \
':(exclude)convex/_generated/**' 2>/dev/null || \
git diff --cached -- . \
':(exclude)*.lock' \
':(exclude)yarn.lock' \
':(exclude)package-lock.json' \
':(exclude)pnpm-lock.yaml' \
':(exclude)composer.lock' \
':(exclude)Podfile.lock' \
':(exclude)Gemfile.lock' \
':(exclude)pubspec.lock' \
':(exclude)*.min.js' \
':(exclude)*.min.css' \
':(exclude)*.map' \
':(exclude)*.snap' \
':(exclude)dist/**' \
':(exclude)build/**' \
':(exclude).next/**' \
':(exclude).nuxt/**' \
':(exclude).expo/**' \
':(exclude).dart_tool/**' \
':(exclude)android/build/**' \
':(exclude)android/.gradle/**' \
':(exclude)ios/build/**' \
':(exclude)ios/Pods/**' \
':(exclude).gradle/**' \
':(exclude)node_modules/**' \
':(exclude)vendor/**' \
':(exclude)__pycache__/**' \
':(exclude)*.generated.*' \
':(exclude)*.g.dart' \
':(exclude)*.freezed.dart' \
':(exclude)convex/_generated/**'`

## Step 2 — Compose the commit message

Arguments passed: `$ARGUMENTS`

Check `$ARGUMENTS` for flags before composing:
- contains `add-scope` → use format `<type>(<scope>): <title>` (infer scope from changed files)
- contains `no-description` → omit the body bullets, title line only
- no flags → use format `<type>: <title>` (no scope by default)

Format (default): `<type>: <title>`
Format (with add-scope): `<type>(<scope>): <title>`

**Type:** `feat` new feature · `fix` bug fix · `refactor` restructure · `chore` build/config/deps/version · `style` UI/formatting · `perf` performance · `docs` docs · `test` tests

**Scope:** infer from the changed files — short lowercase area, e.g. `auth`, `api`, `ui`, `db`, `config`, `notifications`, `settings`, `android`, `ios`

**Title:** max 72 chars · imperative mood · no period · lowercase after colon

**Body:** 2–5 lines using `•` · one blank line after title · each bullet = what changed + why · max 100 chars per bullet

## Step 3 — Stage and commit

Use the message composed in Step 2. If `no-description` or `skip-all` was passed, omit the blank line and body from the commit message entirely.

```bash
git add -A && git commit -m "$(cat <<'EOF'
<type>(<scope>): <title>

<body>
EOF
)"
```

Then run `git rev-parse --short HEAD`.

## Step 4 — Report

If successful:

```
✅ Committed successfully

TITLE:   <first line>
CHANGES: <each bullet, one per line>  ← omit this line if no body was written
COMMIT:  <short hash>
```

If failed:

```
❌ Commit failed
REASON:     <exact git error>
SUGGESTION: <one fix>
```

No extra commentary. Just the report block.
