---
name: git-initial-setup
description: Initialize a git repository and create a comprehensive .gitignore for Next.js, React, React Native, Expo, Express, Node.js, Kotlin, Flutter, and more.
argument-hint: '[commit-all: stage all project files and commit with full title + description]'
disable-model-invocation: true
---

Initialize git and set up .gitignore. Follow each step precisely.

## Step 1 — Check git status

Run both commands:

```bash
pwd
git rev-parse --show-toplevel 2>/dev/null || echo "NONE"
```

Note both values:

- `CURRENT_DIR` = output of `pwd`
- `GIT_ROOT` = output of `git rev-parse --show-toplevel` (or `NONE` if not in any repo)

## Step 2 — Initialize git if needed

Evaluate the three possible states:

**A) `GIT_ROOT` is `NONE`** — no git repo exists anywhere above.
→ Run `git init` and continue to Step 3.

**B) `GIT_ROOT` equals `CURRENT_DIR`** — this folder is already the git root.
→ Skip `git init` and continue to Step 3.

**C) `GIT_ROOT` is a parent of `CURRENT_DIR`** — this folder is nested inside another repo.
→ Stop and ask the user:

> "I found a Git repository already initialized at `[GIT_ROOT]`, which is a parent of your current directory `[CURRENT_DIR]`.
>
> How would you like to proceed?
>
> 1. **Initialize a new repo here** — run `git init` in `[CURRENT_DIR]` (creates a nested/sub-repo)
> 2. **Skip init, just manage .gitignore** — keep using the parent repo, proceed to Step 3"

Wait for the user's choice before continuing:

- Choice 1 → run `git init`, then continue to Step 3
- Choice 2 → continue to Step 3 without running `git init`

## Step 3 — Detect project type

Run this to identify what kind of project this is:

```bash
HAS_PKG=$([ -f "package.json" ] && echo "1" || echo "0")
IS_NEXT=$([ "$HAS_PKG" = "1" ] && grep -q '"next"' package.json && echo "1" || echo "0")
IS_EXPO=$([ "$HAS_PKG" = "1" ] && grep -qE '"expo"' package.json && echo "1" || echo "0")
IS_RN=$([ "$HAS_PKG" = "1" ] && grep -q '"react-native"' package.json && echo "1" || echo "0")
IS_REACT=$([ "$HAS_PKG" = "1" ] && grep -q '"react"' package.json && echo "1" || echo "0")
IS_VITE=$([ "$HAS_PKG" = "1" ] && grep -q '"vite"' package.json && echo "1" || echo "0")
IS_NODE=$([ "$HAS_PKG" = "1" ] && echo "1" || echo "0")
IS_FLUTTER=$([ -f "pubspec.yaml" ] && echo "1" || echo "0")
if [ -f "build.gradle" ] || [ -f "settings.gradle" ] || [ -d "android" ]; then IS_ANDROID="1"; else IS_ANDROID="0"; fi

echo "next=$IS_NEXT expo=$IS_EXPO rn=$IS_RN react=$IS_REACT vite=$IS_VITE node=$IS_NODE flutter=$IS_FLUTTER android=$IS_ANDROID"
```

## Step 4 — Create or update .gitignore

Run this script, which always adds **common** entries and then adds **project-specific** entries based on the detection flags from Step 3:

```bash
GITIGNORE=".gitignore"
[ ! -f "$GITIGNORE" ] && touch "$GITIGNORE"

add_entry() {
  grep -qxF "$1" "$GITIGNORE" || { printf '%s\n' "$1" >> "$GITIGNORE"; ADDED+=("$1"); }
}

ADDED=()

# --- Common (always added) ---
add_entry ".DS_Store"
add_entry "**/.DS_Store"
add_entry "Thumbs.db"
add_entry "desktop.ini"
add_entry ".env"
add_entry ".env*"
add_entry ".env.local"
add_entry ".env.*.local"
add_entry "*.log"
add_entry "logs/"
add_entry ".vscode/"
add_entry ".idea/"
add_entry "*.iml"
add_entry "coverage/"

# --- Node.js / npm / yarn (any JS project) ---
if [ "$IS_NODE" = "1" ]; then
  add_entry "node_modules/"
  add_entry "npm-debug.log*"
  add_entry "yarn-debug.log*"
  add_entry "yarn-error.log*"
  add_entry ".pnp"
  add_entry ".pnp.js"
  add_entry ".yarn/install-state.gz"
  add_entry "*.tsbuildinfo"
  add_entry ".eslintcache"
  add_entry ".cache/"
  add_entry "pids/"
  add_entry "*.pid"
  add_entry ".nyc_output/"
fi

# --- Next.js ---
if [ "$IS_NEXT" = "1" ]; then
  add_entry ".next/"
  add_entry "out/"
fi

# --- Vite ---
if [ "$IS_VITE" = "1" ]; then
  add_entry "dist/"
  add_entry ".parcel-cache/"
fi

# --- React (CRA) — only if not Next.js or Vite ---
if [ "$IS_REACT" = "1" ] && [ "$IS_NEXT" = "0" ] && [ "$IS_VITE" = "0" ]; then
  add_entry "build/"
fi

# --- Expo ---
if [ "$IS_EXPO" = "1" ]; then
  add_entry ".expo/"
  add_entry ".expo-shared/"
fi

# --- React Native / Expo (mobile native) ---
if [ "$IS_RN" = "1" ] || [ "$IS_EXPO" = "1" ]; then
  add_entry "android/build/"
  add_entry "android/.gradle/"
  add_entry "android/app/debug"
  add_entry "android/app/profile"
  add_entry "android/app/release"
  add_entry "ios/build/"
  add_entry "ios/Pods/"
  add_entry "ios/DerivedData/"
  add_entry "*.jks"
  add_entry "*.p8"
  add_entry "*.p12"
  add_entry "*.mobileprovision"
  add_entry "*.orig.*"
  add_entry "local.properties"
fi

# --- Flutter / Dart ---
if [ "$IS_FLUTTER" = "1" ]; then
  add_entry ".dart_tool/"
  add_entry ".flutter-plugins"
  add_entry ".flutter-plugins-dependencies"
  add_entry "*.g.dart"
  add_entry "*.freezed.dart"
  add_entry "build/"
fi

# --- Kotlin / Android (native) ---
if [ "$IS_ANDROID" = "1" ]; then
  add_entry ".gradle/"
  add_entry "local.properties"
  add_entry "*.apk"
  add_entry "*.aab"
  add_entry "*.dex"
  add_entry "*.class"
  add_entry "*.keystore"
  add_entry "gen/"
  add_entry "release/"
fi

echo "Total added: ${#ADDED[@]}"
```

## Step 5 — Commit

Check `$ARGUMENTS`:

**No flag (default)** — commit only `.gitignore`. Use the `ADDED` count and whether `.gitignore` existed before Step 4 to pick the right message:

- `.gitignore` was newly created → `chore: add .gitignore`
- `.gitignore` already existed and entries were added → `chore: update .gitignore`
- `.gitignore` already existed and nothing was added (0 entries added) → skip the commit entirely, nothing to commit

```bash
git add .gitignore && git commit -m "chore: add .gitignore"
# or
git add .gitignore && git commit -m "chore: update .gitignore"
```

**Contains `commit-all`** — stage everything, inspect changes, compose a proper commit message, then commit:

1. Run `git status --short` for a file overview
2. Run `git add -A`
3. Run the diff below to understand what was staged (falls back to `--cached` for first commits with no HEAD):

```bash
git diff HEAD -- . \
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
':(exclude)convex/_generated/**'
```

4. Compose a commit message following Conventional Commits:
   - Format: `<type>: <title>` (no scope)
   - Title: max 72 chars · imperative mood · no period · lowercase after colon
   - Body: 2–5 lines using `•` · one blank line after title · each bullet = what changed + why
   - Use `feat` if this is a new project, `chore` if it's setup/config only
5. Run:

```bash
git commit -m "$(cat <<'EOF'
<type>: <title>

<body>
EOF
)"
```

Then run `git rev-parse --short HEAD`.

## Step 6 — Report

If successful:

```
✅ Git initialized successfully

DETECTED:  <comma-separated list of detected project types>
GITIGNORE: <N> entries added  (or "already up to date")
COMMIT:    <short hash> — <commit title>  (or "skipped — .gitignore already up to date")
CHANGES:   <each bullet, one per line>  ← omit if .gitignore-only commit
```

If failed:

```
❌ Setup failed
REASON:     <exact error>
SUGGESTION: <one fix>
```

No extra commentary. Just the report block.
