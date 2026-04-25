# Git — Version Control, Collaboration, and Testing Workflows

> Subject #15 of the Master Learning Plan.
> Owner: Govind Divekar. Prerequisites: none. Estimated time: ~20 hours.
> Target outcome: Use Git fluently under pressure — recover from mistakes, read any repo's history, drive a PR workflow end-to-end, and wire Git into test automation (hooks, bisect, blame-for-flakes, CI triggers).

---

## 1. Overview & Learning Outcomes

Git is the tool every interview assumes you know. Recruiters rarely quiz you on it — teammates just watch whether you panic when a rebase goes sideways. Aim for the level where you can:

- Explain Git's object model (blobs, trees, commits, refs) without hand-waving.
- Choose between merge / rebase / cherry-pick / revert for a given situation.
- Recover from any local mistake using `reflog` — "Git never loses committed work."
- Resolve conflicts confidently, including during rebases.
- Drive pull requests with a clean, reviewable history.
- Configure hooks and CI to block bad commits.
- Use `bisect` to find the commit that introduced a regression.
- Answer "what's the difference between fetch, pull, and pull --rebase?" without blinking.

---

## 2. Detailed Syllabus

### Part A — Fundamentals (4 h)

1. What Git is vs SVN/CVS; distributed model — 0.5h
2. Installing, configuring (`user.name`, `user.email`, editor, pager) — 0.5h
3. The three trees: working dir, index (staging), HEAD — 1h
4. Object model: blob, tree, commit, tag, refs, HEAD — 1h
5. Creating commits: `add`, `status`, `diff`, `commit`, `log` — 1h

### Part B — Branching & Remotes (4 h)

6. Branches, HEAD, fast-forward vs non-FF merges — 1h
7. Remotes: `fetch`, `pull`, `push`, tracking branches — 1h
8. Merge vs rebase — the mental model — 1h
9. Stashing, cherry-picking, reverting — 1h

### Part C — History Surgery (3 h)

10. Interactive rebase (`rebase -i`) — 1h
11. Amending, squashing, fixup commits — 0.5h
12. `reflog` and recovery — 0.5h
13. `reset` (soft / mixed / hard) vs `revert` vs `checkout`/`switch`/`restore` — 1h

### Part D — Collaboration (3 h)

14. PR workflow: fork/branch, review, squash-merge, rebase-merge — 1h
15. Conflict resolution in practice — 0.5h
16. Branching strategies: GitFlow, trunk-based, GitHub Flow — 1h
17. Signing commits (GPG/SSH), `.gitattributes`, line endings — 0.5h

### Part E — Power Tools (3 h)

18. `bisect` for regression hunting — 0.5h
19. `blame`, `log -S`/`-G` (pickaxe), `log --follow` — 0.5h
20. Hooks: pre-commit, commit-msg, pre-push — 1h
21. Submodules, subtrees, LFS — 0.5h
22. Worktrees — 0.5h

### Part F — CI/CD, QA Workflows & Hygiene (3 h)

23. Git in CI: triggers, shallow clones, sparse-checkout — 0.5h
24. Tag-driven releases, semantic versioning — 0.5h
25. `.gitignore`, secret scanning, leaked-credential recovery — 1h
26. Monorepo patterns, `CODEOWNERS`, branch protection — 0.5h
27. QA-specific: flaky-test triage via blame, bisect on integration failures — 0.5h

---

## 3. Topics — Explanations with Examples

### 3.1 The distributed model

**Concept.** Every clone is a full repo — not a check-out against a server. That means offline commits, cheap branches, and multiple "remotes" if you want them. `origin` is just the conventional name for the remote you cloned from.

```bash
git clone https://github.com/govindd/qa-toolkit.git
cd qa-toolkit
git remote -v                      # list remotes
git remote add upstream https://github.com/company/qa-toolkit.git
```

### 3.2 Configuring Git

```bash
git config --global user.name  "Govind Divekar"
git config --global user.email "govindndivekar@gmail.com"
git config --global init.defaultBranch main
git config --global pull.rebase false          # or true; pick one project-wide
git config --global core.autocrlf input        # mac/linux; 'true' on Windows
git config --global core.editor "code --wait"
git config --global alias.lg "log --graph --oneline --all --decorate"
```

Per-repo overrides live in `.git/config`.

### 3.3 The three trees

**Concept.** Every Git operation moves files between three places:

- **Working directory** — the files you edit.
- **Index** (staging area) — what will be in the next commit.
- **HEAD** — the last commit on the current branch.

```bash
git status
git add file.txt           # working → index
git commit                 # index → HEAD (new commit)
git diff                   # working vs index
git diff --staged          # index vs HEAD
git diff HEAD              # working vs HEAD
git restore file.txt       # discard working changes
git restore --staged f.txt # unstage but keep changes
```

### 3.4 The object model

**Concept.** Git stores four object types:

- **blob** — file contents (no name, no metadata).
- **tree** — a directory; maps names → blobs/trees.
- **commit** — a tree + parents + author + message.
- **tag** — an annotated pointer to a commit (signed, message).

Refs (branches, tags, HEAD) are human-readable names for commit SHAs under `.git/refs/` and `.git/packed-refs`. `HEAD` is usually a symbolic ref to a branch; during rebase/checkout-by-sha it may be "detached."

```bash
git cat-file -p HEAD           # show the commit object
git cat-file -p HEAD^{tree}    # show the root tree
git rev-parse HEAD             # show the SHA
git log --oneline --graph --decorate --all
```

Understanding this single picture makes everything else (rebase, reset, cherry-pick) obvious.

### 3.5 Making commits

**Rules of thumb.**

- One logical change per commit.
- First line ≤ 50 chars, imperative mood ("Fix NPE in OrderService"), blank line, then body ≤ 72 chars per line.
- Reference issue keys (`JIRA-1234`) for traceability.

```bash
git add -p                 # interactive hunk staging — best habit in Git
git commit -m "Fix NPE in OrderService when cart is empty"
git commit --amend         # rewrite the last commit (only if not pushed)
```

### 3.6 Branching

**Concept.** A branch is a movable pointer to a commit. Creating one is O(1).

```bash
git switch -c feature/add-login        # modern form of 'checkout -b'
git branch -a                          # list all
git switch main
git branch -d feature/add-login        # delete merged
git branch -D feature/add-login        # force-delete (unmerged)
```

### 3.7 Merge vs rebase — the mental model

- **Merge** — combines two histories by creating a new "merge commit" with two parents. Preserves the truth that two lines of work happened.
- **Rebase** — replays your commits on top of another branch, producing a linear history. Rewrites SHAs.

```
         A---B---C feature                    A'--B'--C' feature (rebased)
        /                                    /
---X---Y---Z main         →         ---X---Y---Z main
```

**When to use what.**

- Local feature branch, not yet shared → rebase onto main before pushing for a clean history.
- Shared/long-lived branch that others pull → merge, don't rebase. Rebasing shared branches causes force-push chaos.
- `main`/`master` / `release` / anything with branch protection → merge only.

```bash
git switch feature
git fetch origin
git rebase origin/main             # clean linear history for your PR
# or:
git merge origin/main              # preserve merges
```

**`pull --rebase` vs plain `pull`.** `pull` = `fetch` + `merge`. `pull --rebase` = `fetch` + `rebase`. For your own feature branch, `pull --rebase` keeps history tidy; for `main`, both are fine.

### 3.8 Stashing, cherry-picking, reverting

```bash
git stash push -m "wip login"      # save working + index
git stash list
git stash pop                      # apply most recent + drop it
git stash apply stash@{2}          # apply specific

git cherry-pick <sha>              # copy a commit onto current branch
git cherry-pick A^..B              # a range (exclusive start)

git revert <sha>                   # creates a new commit that undoes <sha>
git revert -m 1 <merge-sha>        # revert a merge, keeping mainline 1
```

`revert` is safe on shared history. `reset --hard` is not.

### 3.9 Interactive rebase — history surgery

**Concept.** `git rebase -i` opens an editor with a to-do list of commits. You can `pick`, `reword`, `edit`, `squash`, `fixup`, `drop`, or reorder.

```bash
git rebase -i origin/main          # tidy up your feature branch
# Typical edits:
# pick 1a2b3c4 Add login endpoint
# squash 5d6e7f8 fix typo
# reword 9a0b1c2 -> 'Add integration tests for login'
# drop 3d4e5f6 debug print
```

`--autosquash` pairs perfectly with `commit --fixup <sha>`:

```bash
git commit --fixup 1a2b3c4         # creates "fixup! <subject>"
git rebase -i --autosquash origin/main  # auto-reorders & squashes
```

### 3.10 reset / revert / restore / switch

A reliable mental model:

| Command                 | Moves HEAD | Changes index | Changes working dir |
|-------------------------|:----------:|:-------------:|:-------------------:|
| `git reset --soft <sha>`  | yes      | no            | no                  |
| `git reset --mixed <sha>` (default) | yes | yes     | no                  |
| `git reset --hard <sha>`  | yes      | yes           | **yes (DATA LOSS)** |
| `git revert <sha>`        | creates new commit; no rewrite  |                         |
| `git restore <path>`      | no       | optionally    | yes                 |
| `git switch <branch>`     | yes      | yes           | yes                 |

**Interview trap.** `reset --hard` on a pushed branch + force-push will clobber teammates' work. Use `revert` on shared branches.

### 3.11 reflog — the undo history

**Concept.** Git records every move of `HEAD` and branch tips in `.git/logs/HEAD` for ~90 days by default. *Nothing you committed is truly lost within that window.*

```bash
git reflog                         # list HEAD movements
git reflog show feature            # for a specific branch

# Recover a "lost" branch after a bad reset:
git reset --hard HEAD@{2}          # "two moves ago"
# Or cherry-pick the commit that orphan'd itself:
git cherry-pick 1a2b3c4
```

Teach yourself this now; it's the difference between "I broke it" and "hang on, one sec."

### 3.12 Conflict resolution

**The flow.**

1. `git status` shows files with conflicts.
2. Edit files — pick `<<<<<<<`, `=======`, `>>>>>>>` markers.
3. `git add <file>` to mark resolved.
4. `git commit` (for merge) or `git rebase --continue` (for rebase).

**Better tools.**

```bash
git config --global merge.conflictStyle zdiff3    # shows 3 sides incl. base
git mergetool                                     # if you've set one
git rerere enable                                 # reuse recorded resolutions
```

`rerere` ("reuse recorded resolution") is gold on long-lived branches.

### 3.13 Branching strategies

- **Trunk-based development.** Short-lived branches (< 1 day), everyone merges to `main` multiple times per day, feature flags for incomplete work. Best for CI-heavy teams, GenAI-era speed. Google/Meta/modern startups.
- **GitHub Flow.** `main` is always deployable; feature branches merged via PR. Simple default.
- **GitFlow.** `main` + `develop` + feature/release/hotfix branches. Heavy; made sense in 2010, mostly legacy in 2026.
- **Release branches.** For versioned software (Java libs, mobile apps) — cut `release/1.4.x` at freeze.

### 3.14 PR workflow — from clone to merge

```bash
git switch -c feature/add-login
# ... work ...
git push -u origin feature/add-login
# Open a PR on GitHub

# After review feedback:
git commit --fixup <sha>
git rebase -i --autosquash origin/main
git push --force-with-lease            # safer than --force

# Final: squash-merge vs merge-commit vs rebase-merge are chosen by repo policy
```

**`--force-with-lease`** refuses to overwrite remote if someone else pushed in between. Never use plain `--force` on a shared branch.

### 3.15 Signing commits

**Concept.** Sign commits so reviewers can verify *you* made them.

```bash
# SSH signing (easiest if you already have an SSH key)
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true
```

GitHub will show a green "Verified" badge. Enforce via branch protection: "require signed commits."

### 3.16 .gitignore and .gitattributes

```gitignore
# JVM
target/
.idea/
*.iml

# Node
node_modules/
dist/

# Secrets (never commit!)
.env
*.pem
.aws/

# OS
.DS_Store
Thumbs.db
```

```gitattributes
* text=auto                        # normalize line endings
*.sh text eol=lf
*.bat text eol=crlf
*.png binary
*.jar binary
pom.xml merge=union                # additive merges for POMs
```

### 3.17 bisect — find the commit that broke it

**Concept.** Binary search the history to locate the commit that introduced a bug.

```bash
git bisect start
git bisect bad                     # current commit is bad
git bisect good v1.4.0             # v1.4.0 was good
# Git checks out a midpoint; run your test:
./mvnw -pl api test
git bisect good     # or 'git bisect bad'
# Repeat until Git announces the culprit.
git bisect reset
```

Script it: `git bisect run ./run-repro.sh` — exit 0 = good, 1 = bad, 125 = skip.

### 3.18 blame and pickaxe

```bash
git blame -L 50,80 path/to/File.java    # who last changed lines 50–80
git log -S "OrderService"               # commits that added/removed the string
git log -G "function\s+login"           # same, but regex
git log --follow path                   # follow renames
```

Flaky test triage recipe: `git blame src/test/java/FlakyIT.java` → open the recent commit → read the diff → often the fix is obvious.

### 3.19 Hooks

**Concept.** Scripts under `.git/hooks/` that fire on events. `.git` is not committed — so for team-wide hooks use **`pre-commit` (the framework)** or **Husky** (Node) or **Spotless's `--from-ref`** mode.

```bash
# Install pre-commit framework
pipx install pre-commit
cat > .pre-commit-config.yaml <<'YAML'
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: detect-private-key
YAML
pre-commit install
```

Typical gates to enforce locally: format, lint, secret scan, commit-message regex, unit tests.

### 3.20 Submodules, subtrees, LFS

- **Submodule** — a pointer to another repo at a specific SHA. Great for vendor code; painful for normal sharing.
- **Subtree** — copies another repo's history into a subdirectory. Simpler for consumers, tricky for contributors.
- **LFS** — large binaries stored out-of-band. Use for media, *not* for text-heavy files.

```bash
git submodule add https://github.com/a/b libs/b
git submodule update --init --recursive
git lfs install && git lfs track "*.psd"
```

**Default advice:** avoid submodules unless you really need them.

### 3.21 Worktrees

**Concept.** Multiple working directories backed by one `.git`. Useful when you need to keep a hotfix branch checked out alongside a long-running feature.

```bash
git worktree add ../repo-hotfix hotfix/1.4.1
# do the hotfix in ../repo-hotfix without stashing your feature work
git worktree remove ../repo-hotfix
```

### 3.22 Git in CI

**Speed tricks.**

- **Shallow clone:** `git clone --depth=1` — fastest, but breaks `git log`.
- **Partial clone:** `git clone --filter=blob:none` — blobs fetched on demand.
- **Sparse checkout:** only materialize a subtree — essential for monorepos.

```bash
git clone --filter=blob:none --no-checkout <url> repo
cd repo
git sparse-checkout init --cone
git sparse-checkout set services/api services/domain
git checkout main
```

**GitHub Actions defaults** `fetch-depth: 1`; set `fetch-depth: 0` when you need full history (for `bisect`, `log`, version tools).

### 3.23 Tags and releases

```bash
git tag -a v1.4.0 -m "Release 1.4.0"
git push origin v1.4.0
git tag -d v1.4.0-bad                  # local delete
git push origin :refs/tags/v1.4.0-bad  # remote delete
```

Prefer **annotated** tags (`-a`) over lightweight tags — they carry message + author + date and are signable.

### 3.24 Secret hygiene and leak recovery

**Prevent:**

- `.gitignore` secrets from day one.
- Pre-commit `detect-private-key` / `detect-secrets`.
- GitHub Secret Scanning (free on public; Advanced Security on private).

**Recover (when something leaks):**

1. **Rotate the credential** first — removing from history is *not* enough.
2. Rewrite history with `git filter-repo` (modern) or BFG Repo-Cleaner.
3. Force-push; notify collaborators to re-clone.

```bash
pipx install git-filter-repo
git filter-repo --path secrets.env --invert-paths
```

### 3.25 Monorepo basics

**Techniques.**

- `CODEOWNERS` file for path-based review routing.
- Sparse checkout + partial clone for large repos.
- Branch protection with required checks per-path.
- Tools: Nx, Turborepo (JS), Bazel / Pants (polyglot), Maven reactor (Java).

### 3.26 QA-flavored git workflows

- **Flaky-test sheriff.** `git blame` the test → look at diff → file a fix PR that either fixes the flakiness or marks the test with an `@Disabled("flaky — see JIRA-xxx")` with an owner.
- **Regression detective.** `git bisect run` with a minimal repro script is the fastest way to find the SHA that broke something.
- **Bug repros as tags.** When a customer reports a bug on `v1.4.2`, check out `v1.4.2`, reproduce, write a failing test, then rebase onto `main` and fix.
- **Test data in git.** Keep fixtures in `src/test/resources/fixtures/` with `@DisplayName`-aligned filenames. Track churn with `git log --stat -- src/test/resources/fixtures/`.

### 3.27 Useful aliases

```bash
git config --global alias.lg   "log --graph --oneline --all --decorate"
git config --global alias.st   "status -sb"
git config --global alias.co   "switch"
git config --global alias.cob  "switch -c"
git config --global alias.rbi  "rebase -i"
git config --global alias.pushf "push --force-with-lease"
git config --global alias.last "log -1 HEAD --stat"
git config --global alias.undo "reset --soft HEAD~1"
```

---

## 4. Exercises (Hands-On)

### Beginner

- **A1.** Create a local repo, make 5 commits using `git add -p`, push to a new GitHub repo.
  - *Acceptance:* commits pushed, first-line conventions followed, all ≤ 50 chars subject.
- **A2.** Branch `feature/readme`, edit README, open a PR, self-review, squash-merge.
- **A3.** Create and resolve a deliberate merge conflict between two branches touching the same line.

### Intermediate

- **B1.** Interactive rebase: squash three commits into one, reword the message, drop a debug commit.
- **B2.** Create a `--fixup` commit and use `rebase -i --autosquash` to absorb it.
- **B3.** Simulate a breaking change across 20 commits; use `git bisect run` with a script to find the culprit.
- **B4.** Configure SSH-signed commits; push and see the "Verified" badge on GitHub.
- **B5.** Add pre-commit hooks: trailing-whitespace, detect-private-key, and a local script that runs `./mvnw -q -pl api test`.

### Advanced

- **C1.** Deliberately "lose" a branch with `reset --hard`, recover it via `reflog`.
- **C2.** Accidentally commit a secret, rotate it, then rewrite history with `git filter-repo` and force-push.
- **C3.** Set up a shared monorepo with sparse checkout + `CODEOWNERS` + required checks.
- **C4.** Cherry-pick a hotfix across 3 active release branches (`release/1.3.x`, `1.4.x`, `main`) without duplicating commits (`git cherry-pick -x`).

### Capstone — "QA Git Workflow Playbook"

Assemble a real repo (reuse the Library API from files 06/14) with:

1. Trunk-based branch policy; `main` always deployable; branch protection + `CODEOWNERS`.
2. `.gitignore` + `.gitattributes` + `.editorconfig`.
3. Pre-commit framework with at least 4 hooks including a secret scanner.
4. SSH-signed commits enforced.
5. Squash-merge PR policy, template `.github/PULL_REQUEST_TEMPLATE.md`, commit-message linter (`commitlint` + conventional commits).
6. GitHub Actions triggered on PR with `fetch-depth: 0` for a `release-drafter`-style changelog.
7. `git bisect run` script checked in at `scripts/bisect-api-regression.sh`.
8. README section: "How we use Git on this team" — branching, reviews, releases, rebasing rules.

---

## 5. Additional Reading & Resources

### Books
- *Pro Git*, 2nd ed., Scott Chacon & Ben Straub (free online) — <https://git-scm.com/book>.
- *Git for Teams*, Emma Jane Hogbin Westby.
- *Git from the Bottom Up*, John Wiegley (free) — <https://jwiegley.github.io/git-from-the-bottom-up/>.

### Official docs
- Git Reference — <https://git-scm.com/docs>.
- GitHub Docs → Pull requests, Actions, Codespaces.
- GitLab's Git documentation.

### Interactive learning
- *Learn Git Branching* — <https://learngitbranching.js.org/> (best visual tool for merge/rebase).
- *Oh Shit, Git!?!* — <https://ohshitgit.com/> (recovery recipes).
- *Git Immersion* — <https://gitimmersion.com/>.

### Courses & talks
- Atlassian Git tutorials — <https://www.atlassian.com/git/tutorials>.
- Scott Chacon's "Getting Git Right" talk.
- "So you think you know Git" — Scott Chacon, FOSDEM.

### Tools
- GitHub CLI (`gh`), GitLab CLI (`glab`).
- `tig` — terminal log viewer.
- `lazygit` — TUI for Git.
- `delta` — syntax-highlighted diffs.
- `git-absorb` — automatic fixup commits.

---

## 6. Index

- `add -p` — §3.5
- aliases — §3.27
- `bisect` — §3.17, exercise B3
- `blame` — §3.18
- branching strategies — §3.13
- cherry-pick — §3.8
- conflict resolution — §3.12
- `force-with-lease` — §3.14
- `filter-repo` — §3.24
- hooks (pre-commit) — §3.19
- interactive rebase — §3.9
- LFS — §3.20
- merge vs rebase — §3.7
- monorepo — §3.25
- object model — §3.4
- pickaxe (`-S` / `-G`) — §3.18
- PR workflow — §3.14
- `reflog` — §3.11
- `rerere` — §3.12
- `reset` (soft/mixed/hard) — §3.10
- revert — §3.8
- signing commits — §3.15
- sparse-checkout — §3.22
- stash — §3.8
- submodules / subtrees — §3.20
- tags & releases — §3.23
- three trees — §3.3
- trunk-based development — §3.13
- worktrees — §3.21

---

## 7. References

1. Git Reference Documentation — <https://git-scm.com/docs>
2. *Pro Git*, 2nd ed. — <https://git-scm.com/book>
3. *Git from the Bottom Up* — <https://jwiegley.github.io/git-from-the-bottom-up/>
4. Learn Git Branching — <https://learngitbranching.js.org/>
5. Oh Shit, Git!?! — <https://ohshitgit.com/>
6. Atlassian Git Tutorials — <https://www.atlassian.com/git/tutorials>
7. GitHub Docs — <https://docs.github.com/>
8. Trunk Based Development site — <https://trunkbaseddevelopment.com/>
9. Conventional Commits — <https://www.conventionalcommits.org/>
10. git-filter-repo — <https://github.com/newren/git-filter-repo>
11. pre-commit framework — <https://pre-commit.com/>
12. GitHub CLI — <https://cli.github.com/>
13. Scott Chacon, "Getting Git Right" — Atlassian Summit talk.
14. Linus Torvalds, Git talk at Google (2007, still gold).

---

*End of Subject 15 — Git.*
