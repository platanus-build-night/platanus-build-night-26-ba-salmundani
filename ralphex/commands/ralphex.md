---
description: Iterative dev loop - Claude Code implements, Codex reviews, repeat until clean
argument-hint: Describe the coding task to implement
allowed-tools: ["Read", "Write", "Edit", "Grep", "Glob", "Bash"]
---

# Ralphex: Iterative Development with Codex Code Review

Execute an iterative development loop where you (Claude Code) plan and implement code, then Codex reviews your work. Loop until Codex gives a clean review.

**User's task:** $ARGUMENTS

---

## Step 0: Read Settings

1. Check if `.claude/ralphex.local.md` exists in the project root.
2. If it exists, read it and parse the YAML frontmatter to extract:
   - `base_branch`: The branch to diff against (default: `main`)
   - `codex_model`: The Codex model to use (REQUIRED - if missing, ask the user)
3. If the file does not exist, ask the user:
   - What base branch to diff against (suggest `main`)
   - What Codex model to use (e.g., `o3`, `o4-mini`, `codex-mini`)
   Then create `.claude/ralphex.local.md` with their answers.

4. **Validate settings values**: Both `base_branch` and `codex_model` must contain only alphanumeric characters, hyphens, underscores, dots, and forward slashes (regex: `^[a-zA-Z0-9._/-]+$`). If either value contains other characters, reject it and ask the user for a valid value. This prevents shell injection.

5. **Resolve the review target**: Run `git rev-parse --abbrev-ref HEAD` to get the current branch. Compute a `review_target` value:
   - If the current branch is different from `base_branch`, set `review_target` = `base_branch`.
   - If the current branch equals `base_branch`, check if `origin/{base_branch}` exists by running `git rev-parse --verify origin/{base_branch}`. If it exists, set `review_target` = `origin/{base_branch}`. If it does not exist, inform the user that there is no remote to compare against and stop.
   - Verify there is actually a diff by running `git diff --stat {review_target}...HEAD`. If the diff is empty, inform the user there are no changes to review and stop.

Store `review_target` and `codex_model` for use throughout the workflow. Use `review_target` (not `base_branch`) in all subsequent git and Codex commands.

---

## Step 1: Plan

Analyze the user's task and create a clear implementation plan:

1. Understand the requirements from the task description.
2. If this is the **first iteration**, plan based solely on the user's task.
3. If this is a **subsequent iteration** (after a Codex review), incorporate the review feedback into a revised plan. Focus specifically on addressing the issues Codex raised.
4. Present the plan briefly to the user (do NOT ask for approval - just show what you will do and proceed).

---

## Step 2: Implement

Execute the implementation plan:

1. Write or modify the necessary code files.
2. Make sure the implementation is complete and functional.
3. If you encounter issues, resolve them as part of the implementation.

---

## Step 3: Commit

Create a clean git state for Codex to review:

1. Stage all changed files with `git add` (be specific about which files - do NOT use `git add .` or `git add -A`). **Never stage `.claude/` directory** - it contains local settings that should not be committed.
2. Commit with a descriptive message. Use this format:
   - First iteration: A message describing the initial implementation
   - Subsequent iterations: A message describing what was fixed based on Codex review
   - Always end the commit message with: `Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>`
3. Verify the working tree is clean with `git status`. There should be no uncommitted changes.

**IMPORTANT**: The workspace MUST be completely clean before proceeding to the review step. Codex reviews the committed diff, so any uncommitted changes will be missed.

---

## Step 4: Codex Code Review

Run Codex to review the branch changes:

1. Run this exact command via Bash:

```
codex exec --full-auto -m {codex_model} -o .claude/ralphex-review.txt "Review the changes of this branch against {review_target}. If you find issues, bugs, improvements, or corrections, describe each one clearly. If the code looks good and you have no corrections, respond ONLY with the exact text: LGTM"
```

Replace `{review_target}` and `{codex_model}` with the resolved values from Step 0.

2. Read the file `.claude/ralphex-review.txt` using the Read tool.

---

## Step 5: Evaluate Review

Determine if the review is clean or has corrections:

1. Read the contents of `.claude/ralphex-review.txt`.
2. The review is **an error** if:
   - The file is empty or does not exist (this means Codex failed silently - treat as error, inform the user, and stop)
3. The review is **clean** (no corrections needed) if:
   - The file contains only "LGTM" (or very close variations)
4. The review **has corrections** if:
   - It contains specific issues, bugs, suggestions, or improvements
   - It describes code changes that should be made

### If corrections exist:
- Display the Codex review to the user, prefixed with the iteration number (e.g., "**Iteration 2 - Codex Review:**")
- Delete `.claude/ralphex-review.txt` using Bash: `rm .claude/ralphex-review.txt`
- Go back to **Step 1: Plan** with the review feedback as context
- In the new plan, specifically address each issue Codex raised

### If clean (LGTM):
- Inform the user that Codex approved the implementation
- Display the total number of iterations it took
- Delete `.claude/ralphex-review.txt` using Bash: `rm .claude/ralphex-review.txt`
- You are done!

---

## Important Rules

- **Never skip the commit step.** Codex reviews the committed diff, so the workspace must be clean.
- **Let Codex review against the base branch.** It will determine the diff itself.
- **Track iteration count.** Display it with each review cycle so the user knows progress.
- **Do NOT ask for user approval** between iterations. The loop is automatic. Only stop when Codex gives LGTM.
- **If Codex CLI fails** (command not found, network error, etc.), inform the user and stop. Do not retry automatically.
- **Clean up** `.claude/ralphex-review.txt` after reading it in every iteration, whether the review is clean or not.
