---
name: datainsights-dump-refresh
description: Weekly Cowork task that refreshes six Syracuse University DataInsights exports (sponsored expenditures, awards, proposals, faculty, postdocs, space) into the shared DataInsights Dump folder, using a dedicated "Claude" Chrome profile that downloads into the connected folder. Encodes per-dashboard filters, fiscal-year bounds, row-count verification, and a delete-free stage/verify/rename fail-safe. Runs unattended Mondays ~6am on Duncan Brown's Mac.
version: "1.0"
license: Apache-2.0
maintainer: Duncan Brown, VP for Research, Syracuse University
created: 2026-09-03
updated: 2026-09-03
repository: https://github.com/duncan-brown/research-tasks
---

# DataInsights Dump — Weekly Refresh

> Open-source Cowork task instructions, maintained in the `duncan-brown/research-tasks`
> repository and treated as versioned software: the license travels with the file, and
> provenance is recorded in the Version History below. This artifact encodes rules that
> expire (fiscal-year bounds, a frozen-year list, dashboard view paths); see **Maintenance
> — parameters that expire** and review on the cadence stated below.

## Version Information

| Field | Value |
|---|---|
| **Version** | 1.0 |
| **Created** | 2026-09-03 |
| **Last Updated** | 2026-09-03 |
| **Maintainer** | Duncan Brown, VP for Research, Syracuse University (Syracuse University OSPO) |
| **License** | Apache License 2.0 |
| **Review cadence** | Annually at the SU fiscal-year rollover (July), and whenever a source dashboard's filters, view path, or schema change |

### Version History

- **1.0** (2026-09-03): Initial version-controlled release. Re-architected from the prior ad-hoc instructions to run under a dedicated "Claude" Chrome profile whose download location is a `Downloads` subfolder *inside* the connected DataInsights Dump, so the task needs only the Dump connected — no `~/Downloads` or home-folder access, and personal downloads never enter the Dean-shared folder. Added a Step 0 profile self-verification (probe download must land in `<Dump>/Downloads/`). Replaced the old "move from `~/Downloads` then delete source / delete bad file" steps with a delete-free **stage → verify → rename** fail-safe, because the Cowork FUSE mount blocks `unlink`. Validated end-to-end against live DataInsights and the local sandbox on 2026-09-03. Removed the PhD Progress Tracker handling (dashboard down; to be restored when fixed) and stale dated examples.

### Maintenance — parameters that expire

This task "degrades dangerously rather than quietly": stale values produce confident, well-formatted output against the wrong rules. Review the following at least annually (SU fiscal-year rollover, July) and whenever a dashboard changes:

- **Fiscal-year bounds** in the six-file table: the current-year file (`Expenditure_Details_FY27`, FY From/To = 2027) and the `To = 2027` upper bound on Award/Proposal Details must advance each fiscal year. The FY-defaults warning references specific years and row counts.
- **Frozen-year list** under "Do NOT touch" (`Expenditure_Details_FY17`–`FY26`): extend by one each year as the current year closes.
- **Minimum row-count thresholds**: recalibrate if underlying volumes shift materially (they should trend upward; a large drop is a signal, not a reason to lower the threshold).
- **Dashboard view paths / URLs** and the `Crosstab → Excel` UI flow: Tableau upgrades can move controls or rename views.
- **The dedicated Chrome profile**: if the profile is recreated, re-run Step 0's probe to confirm its download folder still points into `<Dump>/Downloads/`.
- **PhD Progress Tracker**: excluded in 1.0 because the dashboard is down. Restore its row and handling when the maintainer confirms it is fixed.

### Versioning Instructions for Task Updates

1. **Auto-increment the minor version** on routine edits: 1.0 → 1.1 → 1.2, etc.
2. **Update the "Last Updated" date** and the `updated:` frontmatter field to the current date.
3. **Add an entry to Version History** with a brief description of what changed.
4. **Major version updates** (e.g., 1.x → 2.0) — architecture or scope changes — require explicit maintainer instruction.

### GitHub Repository Update Workflow

Maintained in `duncan-brown/research-tasks` under `datainsights-dump-refresh/`.

1. **Commit source changes** to `main` on the `datainsights-dump-refresh/` path; commit message summarizes the change, e.g. `datainsights-dump-refresh v1.1: advance FY bounds to 2028`.
2. **Tag the release** with a lightweight tag on the source commit, named `datainsights-dump-refresh-v{X.Y}` (the `{name}-v{version}` convention avoids tag collisions across tasks in the repo).
3. **Privacy scan before committing**: grep the file for any personally identifying information (real names beyond the named maintainer, salaries, third-party contacts) introduced during editing. This task file should contain none by design.
4. After updating the source, paste the operational body (everything below the horizontal rule) into the Cowork scheduled task's Instructions field so the running task matches the committed source.

### License

This task is licensed under the Apache License, Version 2.0. A `LICENSE` file containing the full Apache-2.0 text sits in the repository root; the license travels with any fork or derivative.

---

Refresh the DataInsights Dump folder with current exports from Syracuse University DataInsights, using the dedicated "Claude" Chrome profile.

## Context and standard of care

This folder feeds analysis skills (oor-dashboards, faculty-by-unit, postdoc-analysis, tririga-space-analysis) and is shared with Dean Behzad Mortazavi (CAS) and Greg Hoke, who run recurring reports from it. Each skill automatically selects the newest file matching its pattern by filename date prefix.

That auto-selection is why a bad file is worse than no file: an empty or truncated export written today immediately supersedes a good older one and silently poisons every downstream query. Never write a file that fails its row-count check. Failing loudly is always the correct outcome.

This is an unattended run (Mondays ~6am, local machine). Execute autonomously; make reasonable choices and note them in the manifest. Take side-effectful actions (downloads) only as this task specifies. When in doubt, produce a report of what you found.

## Environment

- **Connected folder: `DataInsights Dump` ONLY.** Do not connect `~/Downloads`, the home folder, or any other directory. Resolve the sandbox path at runtime (it appears under `/sessions/*/mnt/DataInsights Dump`).
- The dedicated **"Claude" Chrome profile** has its download location set to **`<Dump>/Downloads`** — a subfolder *inside* the connected Dump. All crosstab downloads land there and are therefore readable from the shell. Your default personal Chrome profile still downloads to `~/Downloads` and is never touched.
- Use `mcp__workspace__bash` for ALL file operations. Do NOT use the OneDrive or local-filesystem MCP connectors — every tool on both fails with a JSON Schema dialect error.
- **Delete/unlink is blocked on the mount.** You can create, write, and rename (`mv`), but you cannot delete. Design around this: never rely on removing a bad file. A cross-folder `mv` (e.g. Downloads subfolder → Dump root) falls back to copy because they report different device IDs — that copy is fine; just don't expect the source to be removed.

## Step 0 — Select and verify the Claude browser profile (do this FIRST, before any dashboard)

1. `list_connected_browsers`. If none are connected, STOP and report "Claude Chrome profile not connected at run time" — do not proceed, do not fall back to another profile.
2. `select_browser` on a connected browser. Do NOT hard-code a deviceId — it may change across reboots. If exactly one browser is connected, select it. If more than one is connected, they cannot be told apart by name (all report generically), so run the probe below against each candidate until one passes; if none pass, STOP.
3. **Verify you are driving the right profile with a download-destination probe** (turns "am I on the right Chrome?" into a test, not an assumption):
   - Open a normal page (e.g. `https://example.com`), then via `javascript_tool` trigger a tiny throwaway download (Blob → `a[download='__profile_probe_<timestamp>.txt']` → click).
   - Wait 5s, then check with bash whether the file appeared in `<Dump>/Downloads/`.
   - If YES: correct profile. `mv` the probe into `<Dump>/.sandbox_scratch/` and continue.
   - If NO (landed elsewhere / not visible): wrong profile or misconfigured download folder. STOP and report — do NOT download any real data.

## The six files — no others

Use `Crosstab` -> `Excel` from the toolbar download icon for every one. Stamp each filename with TODAY's date (the download date), format `YYYYMMDD_<Name>.xlsx`. Write a NEW file each week; never overwrite or delete prior versions.

| # | Output | Dashboard | Required filter setup | Min rows |
|---|---|---|---|---|
| 1 | `Expenditure_Details_FY27.xlsx` | [Detailed Sponsored Expenditures](https://datainsights.syr.edu/#/site/data/views/OfficeofResearchDetailedExpenditures/Details?:iid=1) | **Fiscal Year From = 2027, Fiscal Year To = 2027** | 3000 |
| 2 | `Award_Details.xlsx` | [Anticipated & Supplement Amounts](https://datainsights.syr.edu/#/site/data/views/OfficeofResearchAnticipatedandSupplementAmounts/Details) | **Fiscal Year From = 2014, Fiscal Year To = 2027** | 3800 |
| 3 | `Proposal_Details.xlsx` | [Sponsored Proposals](https://datainsights.syr.edu/#/site/data/views/SponsoredProposalsatSyracuseUniversity/Details) | **Fiscal Year From = 2014, Fiscal Year To = 2027** | 8900 |
| 4 | `Faculty_by_Unit.xlsx` | [Faculty by Unit](https://datainsights.syr.edu/#/site/data/views/FacultybyUnit/FacultybyUnit) | **Clear TENURE_STATUS_DESC** — open it and click `(All)`; it defaults to only Tenure + Non Tenure On Track | 2200 |
| 5 | `Postdoc_visa_info.xlsx` | [Postdoc Contact and Visa Info](https://datainsights.syr.edu/#/site/data/views/PostdocContactInfoandCounts_16944584968470/Postdoc_visa_info) | none | 100 |
| 6 | `Tririga_Space_Data.xlsx` | [Tririga Space Data](https://datainsights.syr.edu/#/site/data/views/TririgaSpaceData/TririgaSpaceData) | none | 25000 |

### Critical: the fiscal year defaults are wrong

Award Details and Proposal Details default to **Fiscal Year From = 2022**, not 2014. Downloading on the default silently discards eight years (Award Details yields ~1,549 rows instead of ~3,891) and the file looks completely normal. Always set the From dropdown to 2014 explicitly and confirm it reads 2014 in a screenshot before downloading. Setting From above To produces an empty view, so set From first, wait for the view to reload, then set To.

## Per-file procedure

1. Navigate to the dashboard. Wait 20 seconds; these views are slow. Screenshot to confirm data rendered.
2. Apply the filter setup above. Screenshot to confirm it took effect.
3. Record the "Last Refreshed on ..." timestamp shown in the view header.
4. **Before downloading, verify the view is not empty.** If filters read `(None)`, a dropdown shows "No Items.", or the `Data` download option is greyed out, the extract is empty — do NOT download. Wait 60 seconds and retry once from step 1. If still empty, mark FAILED and move on.
5. Download icon -> `Crosstab` -> confirm the correct sheet checkbox and `Excel`, then Download. Wait 15 seconds. The file lands in `<Dump>/Downloads/`.
6. **Intake:** select the **newest `.xlsx` in `<Dump>/Downloads/` by modification time**, not by name (Chrome appends ` (1)` on name collisions, and prior weeks' raw downloads remain there because delete is blocked).
7. **Stage -> verify -> finalize (delete-free):**
   - a. Copy the newest download to `<Dump>/.incoming_YYYYMMDD_<Name>.xlsx` (hidden staging name; `YYYYMMDD` = today, the download date).
   - b. Open the staged file with pandas. **Strip whitespace from column names** before matching anything — several columns have trailing spaces (`Building  `, `Project ID  `). Assert row count >= the table minimum.
   - c. If it PASSES: `mv` the staged file to `<Dump>/YYYYMMDD_<Name>.xlsx` (same-device rename, allowed). Done.
   - d. If it FAILS: leave it as the hidden `.incoming_` file (you cannot delete it) and mark FAILED. The hidden name matches no skill's glob, so it cannot poison downstream selection.
   - e. Do NOT try to delete the raw download from `<Dump>/Downloads/` (blocked, and harmless — its name doesn't match the skill patterns).

## Do NOT touch

- `Expenditure_Details_FY17` through `FY26` — closed years, permanently frozen.
- `nsf_herd_syracuse_*` — updated manually by Duncan.
- Never delete anything from the destination folder (you can't anyway).

## Manifest — required every run

Append a dated section to `REFRESH_LOG.md` in the Dump. Per file record: output filename, byte size, row count, the dashboard's "Last Refreshed on" timestamp, and OK or FAILED with reason. Note that FY17–FY26 and the NSF HERD files were intentionally skipped. Record both the download date (filename) and the source refresh timestamp (manifest) — the filename says when the file was pulled; only the manifest reveals whether the underlying data was current when pulled.

## Reporting

Lead with failures. If any file failed, say so in the first line. State plainly how many of six succeeded. If the Claude profile wasn't connected, the profile probe failed, or DataInsights presented a login/SSO/MFA prompt that cannot be cleared without interaction, STOP and report that clearly rather than implying the folder was updated. Do not save partial or placeholder files. Do not analyze or interpret the data; this task only refreshes files and logs what it did.

## Operational prerequisite (outside this task)

The "Claude" Chrome profile must be running with the Claude extension connected at run time. Keep that profile open or add it to login items. If it is not connected, Step 0 will stop the run rather than silently use the wrong profile.
