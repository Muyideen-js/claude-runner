# Claude runner

Claude Code fixes your assigned GitHub issues on GitHub's machines, using your
Claude Pro or Max plan. Your computer can be off.

It runs in two ways:

- **From the dashboard.** Choose **Claude (Pro) via GitHub runner** in
  **AI settings**. **Fix** then hands the issue to this runner. The dashboard
  picks up the draft PR, watches CI and sends CI failures back to Claude for
  repair.
- **On its own.** Every 3 hours the runner takes one open issue assigned to
  you that has no solver PR yet, fixes it, and opens a draft PR. It tries each
  issue at most once a day.

## Setup (once, about 5 minutes)

1. **Get a Claude token.** Run this in a terminal where Claude Code is
   installed, sign in with your Claude account, and copy the token it prints:

   ```bash
   claude setup-token
   ```

2. **Create the runner repository** on GitHub, for example
   `your-username/claude-runner`.
   - A **public** repository gets unlimited free Actions minutes. Its run logs
     are public, but they only show public issue text and the change Claude
     made. Your secrets are always hidden.
   - A **private** repository keeps the logs private, but it uses your monthly
     Actions minutes (about 2,000 on the free plan).

3. **Add the workflow.** Copy `.github/workflows/claude-solve.yml` from this
   folder into the new repository, at the same path, and commit it to the
   default branch.

4. **Add two secrets** under *Settings → Secrets and variables → Actions*:

   | Secret | Value |
   |---|---|
   | `CLAUDE_CODE_OAUTH_TOKEN` | the token from step 1 |
   | `FORK_TOKEN` | a GitHub token that can push to your forks and open pull requests (a classic token with `public_repo`, or `repo` for private forks) |

   Optionally add a **variable** named `CLAUDE_MODEL` set to `opus`. The
   default is `sonnet`.

5. **Connect the dashboard.** Open **AI settings** and choose
   **Claude (Pro) via GitHub runner**. Enter `your-username/claude-runner` and
   click **Test connection**, then **Save**. The GitHub account tab you use
   must be able to run this repository's workflows. Your own token does, as
   long as you own the runner repository.

## How it keeps your credentials apart

- Claude, and any code from the issue's repository that Claude runs (tests,
  builds), runs in the `solve` job. That job has your Claude token but never
  your GitHub token.
- The change leaves that job as a patch file.
- A fresh machine in the `push` job applies the patch and pushes it to your
  fork, using your GitHub token.
- Files under `.github/workflows` are never pushed.

## Limits

- **Claude usage limits.** Your plan's usage limits apply. When a run hits
  one, the dashboard waits an hour and tries again. Only one runner job runs
  at a time.
- **Public repositories only.** The runner works on public upstream
  repositories.
- **Not for other people.** Use this for your own issues. Other people should
  set up their own runner with their own Claude plan.
- **Scheduled runs stop on inactive repositories.** GitHub disables scheduled
  workflows in a repository with no activity for 60 days. Re-enable it in the
  **Actions** tab, or push any commit.
