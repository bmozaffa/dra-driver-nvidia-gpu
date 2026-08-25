---
name: dra-proposal-and-design-review
description: Draft or review maintainer-ready proposals, ADRs, design notes, feature-gate plans, and implementation plans for the NVIDIA GPU DRA driver. Use when a change affects user-facing APIs, CRDs, Helm values, ResourceSlice attributes, checkpoint schemas, feature gates, dependencies, upstream coordination, or multi-node ComputeDomain and IMEX behavior.
---

# DRA Proposal and Design Review

## Purpose

Turn a DRA driver idea into a bounded, source-backed design artifact that follows the repository's proposal process and exposes its compatibility, ownership, safety, and test implications.

## Triggers

- Draft or review a proposal, ADR, implementation plan, scope analysis, or maintainer-ready design.
- Add or graduate a feature gate.
- Change a CRD, API field, Helm value, ResourceSlice attribute, CLI flag, checkpoint schema, external dependency, or multi-node coordination behavior.
- Decide whether work belongs in this driver, upstream Kubernetes, GPU Operator, the device plugin, or another layer.

## Audience

- Contributors
- DRA maintainers and reviewers
- Documentation and release owners

## Inputs

- Problem statement, named user or workload, and desired outcome
- Related issues, PRs, upstream work, or release target
- Affected components and user-facing examples
- Upgrade/downgrade, platform, Kubernetes, driver, GPU, and feature-gate constraints
- Available unit, integration, mock, BATS, and real-GPU test environments

## Workflow

1. Read `site/content/contribute/proposals/_index.md` and `site/content/contribute/proposals/NNNN-template.md` completely.
2. Decide whether a proposal is required. Require one for the cases enumerated by the proposal process; explain why a bug fix, docs-only change, or internal refactor can proceed directly when exempt.
3. Verify the problem against current code, repository docs, relevant release notes, Kubernetes DRA docs, and GPU Operator docs. Label public and internal supplemental sources.
4. Establish the layer boundary. State why the change belongs here instead of upstream Kubernetes/DRA, GPU Operator, the legacy device plugin, a runtime, or a separate tool.
5. Start from the repository template. Use the next proposal number only after inspecting existing proposal files; keep status `provisional` until maintainers agree.
6. Write the release-note-quality summary and a concrete user-facing example first.
7. Define measurable goals, explicit non-goals, the smallest valuable slice, affected components, and the single authoritative owner for new state.
8. Cover API and configuration changes, feature-gate stage and graduation, upgrade and downgrade behavior, environment floor, and compatibility with adjacent features.
9. Build a test plan that names appropriate files or jobs across unit, controller/integration, generated artifacts, Helm, mock NVML, BATS, Ginkgo e2e, and real GPU or MNNVL coverage. Explain unavailable hardware coverage.
10. Analyze failure modes including prepare/unprepare races, force deletion, reboot, restart, checkpoint migration, leader election, multi-node partitions, privilege, device mounts, and fail-fast behavior.
11. Compare alternatives, including existing CEL selectors, DRA primitives, current CRD/Helm knobs, upstream work, and reusable NVIDIA libraries.
12. Review for scope, user surface, ownership, upgrade impact, alternatives, drawbacks, open questions, testability, docs, and release/support consequences.
13. Return the design plus a short list of unresolved maintainer decisions. Do not claim an unreviewed proposal is implementable.

## Expected Outputs

- Proposal or ADR draft
- Structured design-review findings
- Scope and layer analysis
- Implementation and validation plan

## References

- Public proposal process: `site/content/contribute/proposals/_index.md`
- Public proposal template: `site/content/contribute/proposals/NNNN-template.md`
- Public architecture: `site/content/docs/concepts/architecture.md`
- Public development workflow: `site/content/contribute/development.md`
- Public upstream repository: https://github.com/kubernetes-sigs/dra-driver-nvidia-gpu
- Public Kubernetes DRA concepts: https://kubernetes.io/docs/concepts/scheduling-eviction/dynamic-resource-allocation/
- Public GPU Operator DRA guide: https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/dra-intro-install.html
- Internal supplemental: none provided by this skill

## Validation

- Include status, authorship context, related work, summary, motivation, goals, non-goals, layer rationale, user-facing example, affected components, state owner, smallest slice, design, upgrade/downgrade, environment floor, test plan, risks, alternatives, drawbacks, and open questions.
- Make each goal imply at least one validation check.
- Name the reviewer roles and source anchors.
- For the scenario "add a new ResourceSlice attribute behind a feature gate", cover proposal necessity, CEL-facing naming, gate stage/default/dependencies, code generation if APIs change, scheduler-version compatibility, docs, release note, and unit plus e2e coverage.
- Flag contradictions between current code, docs, and release assumptions instead of silently choosing one.

## Safety Constraints

- Do not modify clusters, external services, issues, PRs, or releases without explicit approval.
- Preserve internal-only information boundaries in upstream-ready drafts.
- Never include Kubernetes client configuration files, authentication credentials, secrets, customer data, or private cluster identifiers in proposals or design artifacts. Use only minimal redacted examples, and omit sensitive values from diagrams, commands, logs, patches, and upstream-ready drafts.

## Non-Goals

- Do not bypass the repository proposal lifecycle or maintainer approval.
- Do not turn speculative behavior into a compatibility promise.
- Do not implement the design unless the user separately requests implementation.

## Owners

- DRA maintainers
- DRA documentation maintainers

## Refresh Policy

- Review whenever the proposal template or lifecycle changes.
- Review each DRA driver release and after major Kubernetes DRA or GPU Operator changes.
- Revalidate external links and feature-gate conventions before drafting a release-bound proposal.

## Registration Notes

Keep this `SKILL.md` as the shared source of truth. Use `agents/openai.yaml` only as registration metadata.
