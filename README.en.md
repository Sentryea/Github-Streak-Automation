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

To have these commits appear on your GitHub contribution graph, set `GIT_AUTHOR_EMAIL` to an email verified with your GitHub account or to the noreply address shown in your account settings. Contributions from private repositories are also subject to your profile's private-contribution settings.

## Configuration

Optionally add these repository variables under **Settings → Secrets and variables → Actions → Variables**. If you leave them unset, the workflow uses its built-in defaults.

| Variable | Purpose | Example |
| --- | --- | --- |
| `GIT_AUTHOR_NAME` | Display name for automated commits | `octocat` |
| `GIT_AUTHOR_EMAIL` | Email address for automated commits | A verified email shown in your GitHub account settings |

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

Each scheduled run makes a real change to a repository file and pushes a commit. Choose a schedule and log location that suit your use case.
