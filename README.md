# SpaceApp #62: independent Linux verification harness

This repository holds a reviewable patch and a Linux verification workflow for [SpaceApp issue #62](https://github.com/oll4com/spaceapp/issues/62). It is an independent contribution workspace, not the upstream project or a paid service endpoint.

**Status at preparation, 2026-10-23:** this harness has not been published or run. The earlier Windows full `npm test` attempt had 27 failures. Those failures are not reclassified as passes. An actual completed Linux run is needed before reporting Linux results.

## Required payload

Place the complete, reviewed `change.patch` at this repository's root, beside this README and LICENSE. It must apply to exactly:

```text
Repository: https://github.com/oll4com/spaceapp.git
Commit:     0fb6b224a62a4a99ce08917435f3a42b85b514a8
```

Include new source and test files in the patch. Do not include dependencies, build output, runtime state, transcripts, credentials, personal account information or local verification logs. This harness intentionally fails if `change.patch` is missing, empty, does not apply, or contains whitespace errors rejected by `git diff --check`.

Publish only the authorized source patch and these harness files to a **public** repository. The workflow can be started from **Actions → Verify SpaceApp 62 on Linux → Run workflow**. A push to `main` changing the patch or workflow also triggers it. A README-only initial commit does not trigger a run. A private-repository run is skipped and is not verification evidence.

## What the workflow does

The workflow uses the standard GitHub-hosted `ubuntu-latest` runner with `contents: read`. It does not use repository secrets, a PAT, SSH keys, cloud/provider credentials, privileged tokens, self-hosted runners, larger/paid runners, publishing steps or third-party actions. Public Git cloning uses HTTPS without credentials. No `uses:` actions are needed.

1. Fetch the verification repository at its exact workflow `GITHUB_SHA` and print the patch's SHA-256.
2. Fetch SpaceApp at the pinned upstream commit, verify `HEAD`, create a temporary verification branch and run `git apply --check`, then `git apply --index change.patch`.
3. Download the official Node.js **22.23.4 Linux x64** archive and check its pinned SHA-256 before extracting it. Both `node --version` and the archive checksum must match.
4. Run `ELECTRON_SKIP_BINARY_DOWNLOAD=1 npm ci` using the upstream lockfile.
5. Run these unmodified upstream commands in order:

   ```sh
   npm run check
   npm test
   npm run build
   npm run hygiene:preflight
   ```

Each stage preserves its real exit status with Bash `pipefail`; there is no `continue-on-error`. Once installation succeeds, later verification stages are attempted even if an earlier verification stage fails, so failures can be inspected independently. An earlier failed stage still fails the overall job. Timeout, cancellation, skip and infrastructure failure are not passes. The job has a 90-minute maximum.

## Logs and evidence

All command stdout and stderr is visible in the Actions job logs, including failed steps. The final summary lists every stage's actual outcome. Download the job's **log archive** from the Actions UI and record the workflow-run URL, payload commit, printed patch SHA-256, upstream SHA, operating system and Node version when reporting a result.

The workflow also creates temporary tee copies under `$RUNNER_TEMP/spaceapp-62-logs`. These temporary files disappear with the runner. They are **not** artifact uploads. This avoids adding an artifact-storage dependency; the retained Actions console logs are the evidence source. Keep a downloaded log archive if longer retention is needed.

A check/test/build success is not evidence that an integrated GPU meets the frame-time or picking budgets. This hosted CPU workflow does not certify the local browser trace, full-graph loading latency, visual appearance, keyboard operation or WebGL context-loss recovery. Those require their separate, explicitly identified real-device results.

## Hygiene and authorship

`git apply --index` stages the exact patch before dependency installation and checks. Hygiene therefore sees new source files as intentional staged changes, while still rejecting forbidden runtime/generated/report paths. Logs and this verification repository live outside the SpaceApp source directory. Generated files that change tracked content or create forbidden untracked artifacts may make hygiene fail; the workflow does not delete or hide them to obtain a pass.

The verification source checkout remains based on the pinned upstream commit. The workflow does **not** create a source commit, configure a contributor identity, add a `Signed-off-by:` trailer or claim Developer Certificate of Origin compliance. Staging changes is not a commit or a DCO sign-off. Before an upstream contribution is committed, its human author must review the complete patch and apply the project's genuine name/email and sign-off requirements.

## License and pinned sources

SpaceApp is MIT-licensed by **INTRATRADE SINGLE MEMBER P.C.** Its copyright notice and complete license are preserved in [LICENSE](LICENSE). Patch and verification additions are offered under the same MIT terms by their respective authors; no exclusive ownership of upstream code is claimed.

The upstream license was checked at the [pinned upstream LICENSE](https://github.com/oll4com/spaceapp/blob/0fb6b224a62a4a99ce08917435f3a42b85b514a8/LICENSE). The Node archive checksum was checked against the [official Node.js 22.23.4 release](https://nodejs.org/en/blog/release/v22.23.4) and its [SHASUMS256.txt](https://nodejs.org/dist/v22.23.4/SHASUMS256.txt):

```text
1805bcd35e6815559bd2eb97db438d3fdb0eb3a5e6a977b7c1badca76337e9ce  node-v22.23.4-linux-x64.tar.xz
```

Workflow permission and outcome behavior follows the [official GitHub Actions syntax reference](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax). No unpublished action revision or floating action tag is executed by this harness.
