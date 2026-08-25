---
name: dra-release-support-docs-readiness
description: Assess and prepare NVIDIA GPU DRA driver release, support, documentation, known-issue, compatibility, and handoff readiness. Use when asked for a release checklist, changelog or release-note impact, version matrix, docs updates, support caveats, image/chart promotion review, upgrade guidance, or stale-source refresh across the repository and GPU Operator documentation.
---

# DRA Release, Support, and Docs Readiness

## Purpose

Produce a release-scoped readiness record that aligns code, generated artifacts, chart and image versions, automation, documentation, validation, support boundaries, and current external product guidance.

## Triggers

- Prepare or review a release checklist, release PR, changelog, or support handoff.
- Assess documentation, compatibility matrix, known issues, upgrade/downgrade, or release-note impact.
- Verify version surfaces, automation, image/chart promotion, or publication readiness.
- Refresh stale links, source anchors, or GPU Operator integration guidance.

## Audience

- Release shepherds
- Support and documentation owners
- DRA maintainers, QA, and product stakeholders

## Inputs

- Target version, release type, branch, base tag, and commit range
- Included issues/PRs, proposals, user-facing changes, and known issues
- Validation evidence and hardware/provider coverage
- Supported Kubernetes, GPU Operator, NVIDIA driver, runtime/CDI, GPU, and MNNVL matrix
- Publication, promotion, docs, and support owners

## Workflow

1. Establish source precedence and current state:
   - Inspect `VERSION`, `versions.mk`, Helm `Chart.yaml`, `site/hugo.toml`, and the target branch.
   - Read `RELEASE.md`, `.github/workflows/release-automation.yml`, current GitHub releases, and external promotion configuration.
   - When prose and executable automation differ, report the drift and treat the reviewed current workflow as execution authority.
2. Build the change inventory from the exact previous tag/branch range. Group user-facing behavior, APIs/CRDs, feature gates, chart/RBAC/defaults, checkpoints/upgrades, dependencies, docs, tests, and fixes.
3. Verify design state. Link required proposals and confirm their lifecycle status matches implementation and release claims.
4. Audit version surfaces:
   - Root `VERSION` and SemVer format
   - Helm chart `version` and `appVersion`
   - Default image/tag behavior
   - Documentation `driver_version` and `driver_release_tag`
   - Release branch/tag behavior in automation
   - Examples, manifests, upgrade pages, and hard-coded version references
5. Audit compatibility and support claims against the selected release and current official sources. Separate community driver behavior from GPU Operator productized support and known issues.
6. Run or collect the validation matrix from `$dra-build-test-codegen-ci`: generated-code cleanliness, modules, lint, unit, build, Helm, mock, BATS, Go/Ginkgo e2e, upgrade/downgrade, and real GPU/MNNVL coverage. Name untested combinations.
7. Review upgrade safety: CRD application order, Helm naming/values, active ResourceClaims, checkpoint compatibility, rolling components, forward-only transitions, rollback limits, and operator actions.
8. Review docs using the repository information architecture. Update concepts, prerequisites/install/upgrade, task guides, API/gate/Helm/ResourceSlice reference, samples, and contribution docs as required. Build the Hugo site when tooling is available.
9. Prepare support material: known issues, symptoms, diagnostics, workarounds, escalation boundaries, security route, support status, and refresh date. Avoid undocumented compatibility promises.
10. Review release execution and promotion steps without performing them: tracking issue, release PR, automation, tag/branch, staging images/chart, official registry promotion, artifacts, GitHub release, announcements, and post-release docs/support refresh.
11. Require named approvals from DRA engineering, QA, docs, release, and support for their domains. Maintainer approval and required Kubernetes contribution requirements remain external gates.
12. Return a readiness checklist with owner, evidence, status, blockers, assumptions, and last-verified date for each item.

## Expected Outputs

- Release-readiness checklist
- Compatibility and support matrix
- Docs/known-issue gap report
- Support handoff and release-note draft

## References

- Release process: `RELEASE.md`
- Executable release automation: `.github/workflows/release-automation.yml`
- Version/build authority: `VERSION`, `versions.mk`, `deployments/helm/dra-driver-nvidia-gpu/Chart.yaml`
- Documentation release parameters: `site/hugo.toml`, `site/content/contribute/docs.md`
- Development and CI workflow: `site/content/contribute/development.md`
- Upgrade guidance: `site/content/docs/upgrade.md`
- Proposal process: `site/content/contribute/proposals/`
- Public releases: https://github.com/kubernetes-sigs/dra-driver-nvidia-gpu/releases
- Public GPU Operator DRA guide: https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/dra-intro-install.html
- Public GPU Operator release notes: https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/release-notes.html
- Public Kubernetes DRA concepts: https://kubernetes.io/docs/concepts/scheduling-eviction/dynamic-resource-allocation/
- Internal supplemental: none provided by this skill

## Validation

- Tie every release claim to a merged change, proposal, test result, source doc, or explicit owner decision.
- Verify all version surfaces and release-tag links agree or document intentional staging values.
- Verify user-facing API, gate, Helm, checkpoint, prerequisite, and known-issue changes appear in release notes and the correct docs.
- For the ADR scenario "release includes a new alpha GPU feature", include gate/default/support wording, proposal state, upgrade and compatibility impact, chart/docs/samples, tests, known limitations, support escalation, and refresh date.
- Identify stale or contradictory source material instead of copying it into the release.
- Leave publication and external promotion unexecuted unless explicitly authorized.

## Safety Constraints

- Require explicit approval before tagging, branching, pushing, publishing, promoting images/charts, opening or updating issues/PRs, sending announcements, or changing external services.
- Use repository content, public documentation, user-provided material, and approved test fixtures. Keep private operational data out of commands, logs, patches, generated artifacts, and responses.
- Preserve release evidence and avoid destructive local cleanup.

## Non-Goals

- Do not replace release, maintainer, QA, docs, support, security, DCO, or CLA approval.
- Do not infer support from technical possibility or a passing local test.
- Do not publish a release or support statement under readiness assessment alone.

## Owners

- DRA release owners
- DRA support
- DRA documentation maintainers
- DRA engineering and QA

## Refresh Policy

- Review every DRA driver release.
- Refresh after release automation, registry/promotion, versioning, docs structure, Kubernetes support, or GPU Operator lifecycle changes.
- Verify all external support and known-issue sources on the date of each release handoff.
