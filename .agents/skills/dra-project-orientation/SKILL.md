---
name: dra-project-orientation
description: Orient contributors and reviewers to the NVIDIA GPU DRA driver repository, its runtime components, documentation, ownership, and source layout. Use when asked what the project does, how GPU and ComputeDomain paths fit together, where code or docs live, who owns review, or how to onboard without inventing repository paths.
---

# DRA Project Orientation

## Purpose

Navigate the project from a user question to the authoritative component, source, documentation, tests, and owner boundary.

## Triggers

- Explain what this repository does or how NVIDIA GPU DRA differs from the legacy device-plugin model.
- Locate the implementation, configuration, tests, examples, or documentation for a behavior.
- Describe how the five runtime binaries interact.
- Prepare onboarding, issue triage, review routing, or a component map.

## Audience

- New contributors
- Reviewers and maintainers
- Documentation and product contributors
- Support engineers

## Inputs

- The user question and intended audience
- Repository root, branch, tag, or commit
- Relevant issue, PR, release, logs, or workload manifest
- Whether the output is public/upstream-ready or may use internal supplemental material

## Workflow

1. Confirm the checkout and version context with `git status`, `git branch --show-current`, `VERSION`, `go.mod`, and the Helm `Chart.yaml`. Preserve unrelated changes.
2. Read `README.md` and `site/content/docs/concepts/architecture.md` before describing the project.
3. Separate the two resource families:
   - `gpu.nvidia.com`: full GPU, MIG, VFIO, sharing, ResourceSlices, prepare/unprepare, and CDI.
   - `compute-domain.nvidia.com`: ComputeDomains, IMEX daemons and channels, cliques, and MNNVL.
4. Map runtime behavior to the five binaries:
   - `cmd/gpu-kubelet-plugin/`: publish GPU/MIG/VFIO resources and prepare CDI devices.
   - `cmd/compute-domain-kubelet-plugin/`: publish IMEX resources and prepare channel/daemon claims.
   - `cmd/compute-domain-controller/`: reconcile ComputeDomains, per-domain DaemonSets, and claim templates.
   - `cmd/compute-domain-daemon/`: supervise `nvidia-imex` and report peer/readiness state.
   - `cmd/webhook/`: validate NVIDIA opaque configuration.
5. Route data-model questions to `api/nvidia.com/resource/v1beta1/`, generated clients under `pkg/nvidia.com/`, and CRDs under `deployments/helm/dra-driver-nvidia-gpu/crds/`.
6. Route deployment questions to `deployments/helm/dra-driver-nvidia-gpu/`, cluster recipes under `demo/clusters/`, and sample workloads under `demo/specs/`.
7. Route validation questions to colocated Go tests, `test/e2e/`, `tests/bats/`, `hack/ci/`, and `.github/workflows/`.
8. Route documentation by information type:
   - Concepts: `site/content/docs/concepts/`
   - Install, prerequisites, and upgrade: `site/content/docs/`
   - Task guides: `site/content/docs/guides/`
   - API, gates, Helm values, and ResourceSlice fields: `site/content/docs/reference/`
   - Contribution and proposals: `site/content/contribute/`
9. Read `OWNERS` for repository approval roles. Do not infer per-path ownership when no narrower OWNERS file exists.
10. Label the answer's sources as public repository material, internal supplemental material, or both. Keep internal-only facts out of upstream-ready output.
11. Return a concise component map with exact paths and clearly mark any uncertainty or source drift.

## Expected Outputs

- Repository or component map
- Onboarding guide
- Review-routing note
- Source-backed explanation with exact paths

## References

- Public repository: https://github.com/kubernetes-sigs/dra-driver-nvidia-gpu
- Public project documentation: https://dra-driver-nvidia-gpu.sigs.k8s.io/
- Public documentation navigation history: https://github.com/kubernetes-sigs/dra-driver-nvidia-gpu/issues/1103
- Public Kubernetes DRA concepts: https://kubernetes.io/docs/concepts/scheduling-eviction/dynamic-resource-allocation/
- Public productized deployment context: https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/dra-intro-install.html
- Repository anchors: `README.md`, `site/content/docs/concepts/architecture.md`, `site/content/contribute/development.md`, `OWNERS`
- Internal supplemental: none provided by this skill

## Validation

- Verify every named path exists in the selected checkout.
- Identify all five binaries and both resource families without merging their responsibilities.
- Distinguish handwritten APIs from generated clients and CRDs.
- Distinguish local demo flows from GPU Operator deployment guidance.
- For the scenario "where does a GPU ResourceClaim become a runtime device?", trace scheduler allocation to kubelet prepare in `cmd/gpu-kubelet-plugin/` and CDI injection without claiming that the controller performs GPU allocation.
- Include the checkout version or commit when behavior may differ by release.

## Safety Constraints

- Use read-only inspection by default.
- Ask for explicit approval before cluster mutation, destructive local actions, or external-service changes.
- Use repository content, public documentation, user-provided material, and approved test fixtures. Keep private operational data out of commands, logs, patches, generated artifacts, and responses.

## Non-Goals

- Do not replace maintainer review or ownership decisions.
- Do not treat planned behavior, issue discussion, or stale docs as implemented behavior.
- Do not make code, deployment, issue, PR, or release changes under orientation alone.

## Owners

- DRA engineering
- DRA documentation maintainers

## Refresh Policy

- Review every DRA driver release.
- Review after component moves, documentation restructuring, repository transfer, or Kubernetes minor-version updates.
- Recheck external URLs and ownership data before publishing onboarding material.
