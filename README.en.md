# GitHub Streak Automation

[简体中文](README.md) | [English](README.en.md)

Use GitHub Actions to update a log file in your repository and push a commit every day—no scheduled task on your own computer required.

[![Daily workflow](https://github.com/Sentryea/Github-Streak-Automation/actions/workflows/daily-commit.yml/badge.svg)](https://github.com/Sentryea/Github-Streak-Automation/actions/workflows/daily-commit.yml)
![Python 3.11+](https://img.shields.io/badge/Python-3.11%2B-3776AB?logo=python&logoColor=white)

## Quick start

1. Fork this repository, or copy the project files into your own GitHub repository.
2. Make sure `.github/workflows/daily-commit.yml` is on your repository's **default branch** and GitHub Actions is enabled.
3. Open **Settings → Actions → General → Workflow permissions**, select **Read and write permissions**, and save. The workflow needs write access to commit and push the log.
4. Open **Actions → Daily GitHub Streak Commit → Run workflow** and run it once to check your setup.

The workflow will run on its schedule after that. If your default branch is protected, adjust its rules to allow this workflow to push commits.

## How it works

- **Runs daily:** At 08:10 China Standard Time (UTC 00:10).
- **Updates the log:** Appends a dated, timestamped entry to `activity/streak.md`.
- **Avoids duplicates:** Skips a date that already has an automated entry. You can select `force=true` when running the workflow manually to add another entry.
- **Commits changes:** Creates and pushes a commit only when the log file changes.

To have these commits appear on your GitHub contribution graph, `GIT_AUTHOR_EMAIL` must be an address that resolves to your account. Use the **ID-based noreply form** `<numeric-id>+<username>@users.noreply.github.com` (for example `100594650+Sentryea@users.noreply.github.com`; find your ID at https://github.com/settings/users). This form keeps working after you **change your GitHub username**, while the legacy `<username>@users.noreply.github.com` breaks with the old name and drops your commits off the contribution graph. Contributions from private repositories are also subject to your profile's private-contribution settings.

## Configuration

Optionally add these under **Settings → Secrets and variables → Actions**, either as **Variables** or as **Secrets** — the workflow reads Variables first, then Secrets. If you leave them unset, the workflow uses its built-in defaults.

| Variable | Purpose | Example |
| --- | --- | --- |
| `GIT_AUTHOR_NAME` | Display name for automated commits | `octocat` |
| `GIT_AUTHOR_EMAIL` | Email address for automated commits | `100594650+octocat@users.noreply.github.com` |

When `GIT_AUTHOR_EMAIL` is unset, the workflow falls back to `${{ github.actor_id }}+${{ github.actor }}@users.noreply.github.com` — the ID-based noreply form, which keeps commits attributed after a username change.

The schedule, log path, and time zone are set in `.github/workflows/daily-commit.yml`:

- `cron: "10 0 * * *"` — daily at 00:10 UTC, or 08:10 in China Standard Time.
- `STREAK_FILE: activity/streak.md` — path to the log file.
- `STREAK_TIMEZONE: Asia/Shanghai` — time zone used to determine the log's date.

GitHub Actions interprets `schedule` cron expressions in UTC. When changing the schedule, convert your desired local time to UTC.

## Run manually or locally

To start a manual run, open **Actions → Daily GitHub Streak Commit → Run workflow**:

- `force=false` skips the update if an automated entry already exists for today.
- `force=true` appends another entry and commits it, even if today's entry already exists.

The project uses only the Python standard library. It requires Python 3.11 or later. From the repository root, run:

```bash
# Update today's log
python scripts/update_streak.py

# Append another entry even if today's entry already exists
python scripts/update_streak.py --force

# Run the tests
python -m unittest discover -s tests
```

You can also set the log file and time zone with command-line options or environment variables:

```bash
python scripts/update_streak.py --file activity/streak.md --timezone Asia/Shanghai
```

The corresponding environment variables are `STREAK_FILE`, `STREAK_TIMEZONE`, and `FORCE_UPDATE`.

## Project structure

```text
.
├── .github/workflows/daily-commit.yml  # Scheduled workflow and commit settings
├── activity/streak.md                  # Daily log updated by the workflow
├── scripts/update_streak.py            # Log update script
├── tests/test_update_streak.py         # Unit tests
└── pyproject.toml                      # Python project configuration
```

## Troubleshooting

**The workflow succeeded, but there is no new commit.** Check whether today's automated entry is already in the log. By default, the workflow skips duplicate dates. To add another entry, run it manually with `force=true`.

**The push failed with a permission error.** Check that **Settings → Actions → General → Workflow permissions** is set to **Read and write permissions**, and that your default branch's protection rules allow the workflow to push.

**The commit does not appear on your contribution graph.** Check that `GIT_AUTHOR_EMAIL` is associated with your GitHub account and that the commit is on the default branch. For private repositories, also check your profile's private-contribution settings.

**Your contribution streak broke after changing your GitHub username.** GitHub resolves commit authorship by email, so a legacy noreply address (`<old-username>@users.noreply.github.com`) dies with the old name and your commits stop counting. Fix it in two steps: point `GIT_AUTHOR_EMAIL` at the ID-based form (`<numeric-id>+<new-username>@users.noreply.github.com`) so future commits stay attributed, then rewrite the author email of existing history to bring the missing days back.

`git filter-repo` rewrites history while **preserving each commit's original author date**, so GitHub credits the commits to the days they were actually made:

```bash
git filter-repo --force --commit-callback '
commit.author_name = b"your-username"
commit.author_email = b"numeric-id+your-username@users.noreply.github.com"
commit.committer_name = b"your-username"
commit.committer_email = b"numeric-id+your-username@users.noreply.github.com"
'
git push --force-with-lease origin main
```

Back up first (`git branch backup/pre-identity-rewrite && git push origin backup/pre-identity-rewrite`) and delete the backup once you have verified the result.

Each scheduled run makes a real change to a repository file and pushes a commit. Choose a schedule and log location that suit your use case.
