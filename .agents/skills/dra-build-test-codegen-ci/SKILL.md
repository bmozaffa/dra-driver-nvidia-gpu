---
name: dra-build-test-codegen-ci
description: Select, run, and interpret the NVIDIA GPU DRA driver's local build, formatting, lint, code generation, unit, Helm, BATS, Ginkgo e2e, mock-NVML, and CI checks. Use when preparing a PR, validating a patch, reproducing CI, debugging a failed check, or mapping changed files to the minimum credible test set.
---

# DRA Build, Test, Codegen, and CI

## Purpose

Map a repository change to the smallest credible validation set, run safe local checks, and report exact evidence and unrun coverage.

## Triggers

- Ask how to build, test, lint, generate, or prepare a PR.
- Debug a local or CI failure.
- Determine which tests apply to a code, API, chart, docs, dependency, GPU, or ComputeDomain change.
- Validate whether a patch is review-ready.

## Audience

- Contributors
- Reviewers
- DRA engineering and QA

## Inputs

- Worktree diff and target branch
- Affected components, APIs, chart, docs, or dependencies
- Available tools, container engine, cluster, GPU type, and MNNVL capability
- CI job, failure log, and artifact links when debugging
- Approval status for invasive cluster tests

## Workflow

1. Inspect `git status --short`, the full diff, `Makefile`, `site/content/contribute/development.md`, `.github/workflows/`, and the PR checklist. Preserve unrelated changes.
2. Classify the change and choose checks:

   | Change | Required baseline |
   |---|---|
   | Go implementation | Focused package test, `make build`, `make test`, `make check` |
   | API or CRD | Focused tests, `make generate`, `make check-generate`, `make test`, `make check` |
   | `go.mod` / `go.sum` / vendoring | `make check-modules` and the devel-module check used by CI |
   | Helm values/templates/RBAC/CRDs | `make helm-lint`, targeted `helm template`, applicable tests |
   | GPU allocation behavior | Unit tests plus approved `make bats-gpu` or `make test-e2e`; identify real-GPU gaps |
   | ComputeDomain behavior | Unit/controller tests plus approved `make bats-cd`; identify MNNVL/IMEX gaps |
   | Docs | Check links/paths/shortcodes and build the Hugo site when tooling is available |
   | Release workflow | Validate version surfaces and inspect `.github/workflows/release-automation.yml` |

3. Install codegen tools only through the repository workflow (`make -C deployments/devel install-tools`) when needed. Do not invent tool versions.
4. Run focused tests first, then broader checks. Capture the failing command, first actionable error, and relevant artifact—not only the final exit code.
5. Treat `make generate`, `make check-generate`, `make check`, `make check-modules`, and formatting targets as potentially worktree-modifying. Record the pre-run diff and inspect the post-run diff.
6. Use containerized `docker-*` targets only when the development image path is appropriate and dependencies are unavailable locally.
7. Before BATS, read `tests/bats/README.md` and `tests/bats/Makefile`. Confirm the current `kubectl` context, relevant `TEST_*` settings, chart/image source, and explicit approval; the suite is invasive.
8. Before Go/Ginkgo e2e, read `test/e2e/README.md`. Confirm GPU Operator, DRA driver, ResourceSlices, and the target context; record `$ARTIFACTS`.
9. Separate GitHub Actions checks from external Prow/Lambda/GCP GPU coverage. Do not report unit or mock-NVML success as real-hardware coverage.
10. Re-run the smallest failing test after a fix, then the required broader gate.
11. Report passes, failures, skips, environment limits, worktree changes caused by tools, and recommended CI or hardware follow-up.

## Expected Outputs

- Change-to-test matrix
- Local validation report
- CI failure diagnosis
- Review-readiness checklist

## References

- Public development guide: `site/content/contribute/development.md`
- Build and codegen authority: `Makefile`, `common.mk`, `versions.mk`, `deployments/devel/`
- PR checklist: `.github/PULL_REQUEST_TEMPLATE.md`
- GitHub Actions: `.github/workflows/`
- BATS guide: `tests/bats/README.md`
- Go/Ginkgo e2e guide: `test/e2e/README.md`
- External GPU harnesses: `hack/ci/`
- Public repository: https://github.com/kubernetes-sigs/dra-driver-nvidia-gpu
- Internal supplemental: none provided by this skill

## Validation

- Confirm each selected command exists in the current checkout before recommending it.
- Confirm generation and module checks leave only expected diffs.
- Confirm test evidence names the package, suite, scenario, and environment.
- For the scenario "change a field under `api/nvidia.com/resource/v1beta1`", require code generation, generated-diff inspection, focused validation tests, `make check-generate`, unit tests, and relevant Helm/docs checks.
- For the scenario "change only controller reconciliation", avoid claiming GPU allocation e2e coverage unless that path was actually exercised.
- Mark every skipped gate and why it was skipped.

## Safety Constraints

- Ask for explicit approval before BATS, e2e, cluster creation, chart install/upgrade, or any other cluster mutation.
- Never clean, reset, or overwrite unrelated work to make a check pass.
- Do not expose kubeconfigs, credentials, cloud project data, or sensitive logs.

## Non-Goals

- Do not equate a passing build with behavioral correctness.
- Do not require every expensive suite for every change; justify the test boundary.
- Do not modify CI, open PRs, or trigger external jobs unless separately requested and authorized.

## Owners

- DRA engineering
- DRA QA

## Refresh Policy

- Review after Make target, codegen, test-suite, CI-provider, Go, or Kubernetes dependency changes.
- Review every DRA driver release.
- Recheck the PR template and workflows before publishing a readiness checklist.

## Registration Notes

Keep this `SKILL.md` as the shared source of truth. Use `agents/openai.yaml` only as registration metadata.
