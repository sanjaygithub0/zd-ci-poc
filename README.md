# Live PoC — Zendesk/ultimateai CI command injection

Proves that attacker-controlled `CHANGELOG.md` content is executed as shell on a
GitHub-hosted deploy runner by `ultimateai/workflows`.

## One-time setup (about 5 minutes)

1. Create a **public** GitHub repo, e.g. `zd-ci-poc`.
2. Create a **fine-grained PAT** limited to that repo with `Contents: Read-only`
   (Settings -> Developer settings -> Fine-grained tokens).
   Add it as a repository secret named `ULTIMATE_GITHUB_PAT`
   (Settings -> Secrets and variables -> Actions -> New repository secret).
3. Add a repository **variable**
   `AVAILABLE_REGIONS_FOR_DEPLOYMENT` = `europe`
   (Settings -> Secrets and variables -> Actions -> Variables).
4. Upload the contents of this folder to the repo, then edit
   `.github/workflows/poc.yml` and replace every `OWNER/REPO` with your
   `user/zd-ci-poc`.
5. Create a release/tag `v1.0.0` (Releases -> Draft a new release -> tag
   `v1.0.0` -> Publish). This is needed by `fetch-latest-release`.

## Run

1. Actions -> **zd-ci-poc** -> **Run workflow**.
2. Open the run and the **Changelog content** / **filter_changelog** step log.
3. Expected output:

```
ZD_INJECTION_CONFIRMED
uid=1001(runner) gid=1001(runner) groups=1001(runner),...
<runner-hostname>
```

The payload only runs `echo`/`id`/`hostname`; it does not read or send secrets.
Steps after the injection may fail (missing Google Cloud credentials); ignore
them — execution is already proven in the changelog step.

## If the run stops early

- `Region ... is not yet supported` -> step 3 variable is missing.
- `fetch_latest_release` fails -> no release; do step 5.
- k8s checkout fails -> the PAT secret is missing; do step 2.

## What to attach to the Bugcrowd report

- The public repo URL.
- The workflow run URL.
- Screenshot/log of the `uid=` line.
