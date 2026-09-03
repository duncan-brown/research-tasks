# Installing `datainsights-dump-refresh` in Claude Cowork

Unlike a **skill** — which you install by downloading a packaged `.skill` file from a repo's `dist/` and adding it in the app — a **Cowork task** has no package. You configure it by hand in the Claude desktop app: you paste the task's instructions into a scheduled task and set its folder, model, and schedule. This file gives the exact configuration for `datainsights-dump-refresh`, plus how to update it when a new version ships.

There is a hard prerequisite: the dedicated Chrome profile described in [`README.md`](README.md) §5 must already be set up. Without it, the task stops at Step 0 on every run.

---

## Prerequisites

1. **Claude desktop app with Cowork**, latest version, on a paid plan (Pro/Max/Team/Enterprise).
2. **The dedicated "Claude" Chrome profile**, fully configured per [`README.md`](README.md) §5:
   - a dedicated bot Google account signed into the profile;
   - the Claude ("Claude in Chrome") extension installed **and signed into the same Anthropic account your desktop app uses** (via SU SSO);
   - the profile's **download location set to `…/DataInsights Dump/Downloads`**, with "Ask where to save each file" **off**;
   - **DataInsights logged in** in that profile (SSO).
3. **The DataInsights Dump folder synced locally** (OneDrive) so it can be connected to the task.
4. **Access to this repository** to get `TASK.md`.

## Step 1 — Get the task instructions from the repo

Clone or pull the repo, or download the raw file:

```
git clone git@github.com:duncan-brown/research-tasks.git
# or, raw:
# https://raw.githubusercontent.com/duncan-brown/research-tasks/main/datainsights-dump-refresh/TASK.md
```

Open `datainsights-dump-refresh/TASK.md`. It has **two parts separated by a horizontal rule (`---`)**:

- a **metadata header** (frontmatter, Version Information, Version History, etc.) — this is repository bookkeeping, **not** runtime instructions;
- the **operational body** — everything **below** the `---`, starting at "Refresh the DataInsights Dump folder…".

**Only the operational body goes into Cowork.** Note the `version:` value from the frontmatter so you know which version you're installing.

## Step 2 — Create the scheduled task in Cowork

In the Claude desktop app, open Cowork and create a new scheduled task. Set the fields as follows:

| Field | Value |
|---|---|
| **Name** | `Datainsights dump weekly refresh` |
| **Description** | `Weekly refresh of six DataInsights exports into the DataInsights Dump folder` |
| **Instructions** | Paste the **operational body** of `TASK.md` (everything below the `---`). Do **not** paste the metadata header. |
| **Folder** | Connect **`DataInsights Dump` only** (the folder selector → Add/choose folder). Do **not** connect `~/Downloads`, your home folder, or anything else. |
| **Model** | A capable model — **Opus** (the task drives a browser, verifies data, and writes files; use the strongest model available). |
| **Schedule** | **Weekly · Monday · ~6:00 AM.** This is a local task ("Only on this computer") because it needs the local folder and Chrome profile. |

Save the task.

## Step 3 — First run and verification

1. Make sure the **dedicated Chrome profile is open** and its extension is connected (leave it running / add it to macOS login items so the 6am run finds it).
2. Trigger a **manual run** ("Run now").
3. A successful run should:
   - pass **Step 0** (the profile probe download lands in `<Dump>/Downloads/`);
   - write six `YYYYMMDD_*.xlsx` files into the DataInsights Dump;
   - append a new dated section to `REFRESH_LOG.md` in the Dump;
   - end with a report that **leads with any failures** and states how many of six succeeded.
4. If the report says the Claude profile wasn't connected, or DataInsights showed an SSO/MFA prompt, fix the profile/SSO (README §5) and run again. The task is designed to **stop and report** rather than write empty files.

## Updating to a new version

Cowork tasks **do not auto-sync** from GitHub. Updating is a manual copy/paste:

1. `git pull` (or re-download the raw `TASK.md`). Check the new `version:` and read the **Version History** entry to see what changed — pay attention to any **Maintenance — parameters that expire** notes (e.g., fiscal-year bumps).
2. In Cowork, open the existing task and **Edit**.
3. **Replace the entire Instructions field** with the new **operational body** (everything below the `---`).
4. Confirm the connected folder is still **only** `DataInsights Dump` and the schedule is unchanged.
5. Save. Optionally **Run now** to confirm the new version works.
6. Record which version you installed (the app doesn't display it) — e.g., note it in your own log, since the running task text and the repo `version:` are only kept in sync by you.

## Troubleshooting — what the run report will tell you

- **"Claude Chrome profile not connected at run time"** — the dedicated profile isn't open/connected. Open it; add to login items.
- **SSO / MFA prompt** — the DataInsights session expired. Re-log into DataInsights in the dedicated profile.
- **Probe landed in the wrong place (Step 0 stop)** — the profile's download folder isn't `<Dump>/Downloads`. Fix it in that profile's Chrome settings.
- **A file marked FAILED (row count below minimum)** — it's left as a hidden `.incoming_…` file and the good older file is untouched. Investigate the dashboard (empty view, wrong filter, or a real data drop).
- **Folder not updated at all** — confirm the task is connected to the Dump, the Mac was awake at 6am with the desktop app running, and the Chrome profile was connected.

---

*Task maintained in `duncan-brown/research-tasks`. See [`TASK.md`](TASK.md) for the runnable instructions and [`README.md`](README.md) for the architecture. Licensed under Apache-2.0.*
