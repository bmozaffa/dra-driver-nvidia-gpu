---
name: dra-api-crd-feature-gate-change
description: Plan, implement, review, or validate NVIDIA GPU DRA driver API, CRD, opaque configuration, ResourceSlice attribute, Helm value, CLI flag, checkpoint schema, feature-gate, and compatibility changes. Use when work touches api/nvidia.com, generated CRDs or clients, validation, DeviceClasses, public configuration, feature-gate stages or dependencies, or upgrade and downgrade behavior.
---

# DRA API, CRD, and Feature Gate Change

## Purpose

Carry a user-visible or persisted contract change through design alignment, authoritative source edits, generated artifacts, compatibility analysis, documentation, and validation.

## Triggers

- Add or change a CRD, API field, opaque config type, validation rule, DeviceClass, ResourceSlice attribute, Helm value, or CLI flag.
- Add, remove, graduate, default, compose, or constrain a feature gate.
- Change kubelet-plugin checkpoint data or migration behavior.
- Review backward compatibility, Kubernetes resource API versions, or release-note impact.

## Audience

- DRA contributors and maintainers
- Release owners
- API and compatibility reviewers

## Inputs

- Proposed behavior and user-facing example
- Target release and supported Kubernetes versions
- Affected resource family and binaries
- Upgrade/downgrade and stored-state expectations
- Adjacent feature gates and hardware/runtime constraints

## Workflow

1. Inspect the worktree and preserve unrelated changes. Record the target branch, `VERSION`, Kubernetes module versions in `go.mod`, and chart version.
2. Read the proposal process. Require design alignment for any user-facing API, gate, Helm value, ResourceSlice attribute, checkpoint, dependency, or significant behavior change.
3. Identify each contract and its authority:
   - Handwritten API types and validation: `api/nvidia.com/resource/v1beta1/`
   - Feature definitions/defaults/dependencies: `pkg/featuregates/featuregates.go`
   - Generated deepcopy, clients, listers, informers, and CRDs: generated outputs under `api/`, `pkg/nvidia.com/`, and `deployments/helm/dra-driver-nvidia-gpu/crds/`
   - Helm defaults and rendering: `deployments/helm/dra-driver-nvidia-gpu/values.yaml` and `templates/`
   - Admission validation: `cmd/webhook/`
   - Kubelet checkpoints and migrations: checkpoint files in both kubelet-plugin command directories
   - Public reference docs: `site/content/docs/reference/`
4. Search all readers, writers, defaults, validators, serializers, tests, templates, examples, and docs before editing. Identify the single authoritative state owner.
5. Define compatibility:
   - Preserve wire names and semantics unless a reviewed migration exists.
   - Use optional fields and safe zero values for additive checkpoint changes; document old/new binary behavior.
   - Account for the Kubernetes DRA API level used by each supported Kubernetes minor.
   - Treat ResourceSlice attribute names and CEL access paths as public contracts.
   - State CRD update ordering and Helm's CRD upgrade limitations.
6. For a feature gate, define stage, default, introduction version, dependencies, mutual exclusions, composition with adjacent gates, graduation evidence, and removal plan. Compare the code registry with the feature-gate docs and report drift.
7. Implement handwritten sources first. Never hand-edit generated files as the source of truth.
8. Regenerate with the repository workflow. Review the generated diff for unintended schema, default, required-field, RBAC, or client changes.
9. Update Helm values/templates, webhook validation, DeviceClasses, examples, API/feature-gate/Helm/ResourceSlice docs, upgrade guidance, and PR release notes as applicable.
10. Run the change-appropriate validation from `$dra-build-test-codegen-ci`, including focused unit tests, `make generate`, `make check-generate`, `make test`, `make check`, and `make helm-lint` where relevant.
11. Add upgrade/downgrade and negative tests. Include real-cluster coverage only after explicit approval.
12. Return a contract-change summary, compatibility matrix, generated-file list, validation evidence, docs/release impact, and open risks.

## Expected Outputs

- API or feature-gate change plan
- Implementation patch and generated artifacts
- Compatibility and migration matrix
- Review findings and release-note text

## References

- Public API reference: `site/content/docs/reference/api.md`
- Public feature gates: `site/content/docs/reference/feature-gates.md`
- Public ResourceSlice attributes: `site/content/docs/reference/resourceslice-attributes.md`
- Public Helm values: `site/content/docs/reference/helm-values.md`
- Public proposal process: `site/content/contribute/proposals/_index.md`
- Public Kubernetes DRA concepts: https://kubernetes.io/docs/concepts/scheduling-eviction/dynamic-resource-allocation/
- Public Kubernetes API reference for ResourceSlice: https://kubernetes.io/docs/reference/kubernetes-api/resource/resource-slice-v1/
- Internal supplemental: none provided by this skill

## Validation

- Verify handwritten and generated sources agree.
- Verify API validation accepts supported old objects and rejects new invalid combinations.
- Verify `make check-generate` leaves no unexplained generated diff.
- Verify Helm defaults, template validation, binary flags, and feature-gate docs agree with code.
- Verify upgrade, rollback, in-flight claim, restart, and checkpoint behavior is explicit.
- For the scenario "add an optional field to `GpuConfig`", trace it through Go types, validation, webhook decoding, plugin consumption, generated artifacts, docs, samples, unit tests, and release notes.
- Report any existing drift encountered; do not normalize it silently outside the requested scope.

## Safety Constraints

- Inspect the current diff before and after generation; never discard unrelated work.
- Ask before running live-cluster tests or applying CRDs, charts, or workloads.
- Do not delete or rewrite checkpoint files as a troubleshooting shortcut without an approved recovery plan.
- Never expose secrets, kubeconfigs, credentials, or customer data.

## Non-Goals

- Do not bypass proposal or maintainer review for public contracts.
- Do not promise support solely because code compiles.
- Do not manually edit generated outputs in place of changing their source.

## Owners

- DRA engineering
- DRA API and release reviewers

## Refresh Policy

- Review every DRA driver release and Kubernetes dependency minor bump.
- Review after CRD/codegen tool changes, feature-gate lifecycle changes, checkpoint format changes, or GPU Operator compatibility updates.
- Reconcile code and public reference docs before relying on a gate list or default.

## Registration Notes

Keep this `SKILL.md` as the shared source of truth. Use `agents/openai.yaml` only as registration metadata.
