---
name: dra-helm-demo-sample-workloads
description: Plan, render, install, upgrade, or validate NVIDIA GPU DRA driver Helm deployments, local kind or nvkind demos, and GPU or ComputeDomain sample workloads. Use when asked for chart values, prerequisites, demo clusters, quickstarts, sample manifests, DeviceClasses, ResourceSlices, or basic allocation validation, while separating standalone community flows from GPU Operator-managed deployment.
---

# DRA Helm, Demos, and Sample Workloads

## Purpose

Produce a version-aware, reviewable deployment or demo procedure with an explicit plan/apply boundary and observable success criteria.

## Triggers

- Install, upgrade, render, or configure the Helm chart.
- Create a kind, nvkind, GKE, or OpenShift demo environment.
- Run or adapt a sample GPU, MIG, VFIO, sharing, or ComputeDomain workload.
- Validate driver pods, DeviceClasses, ResourceSlices, claims, or CDI/IMEX injection.

## Audience

- Contributors
- Documentation authors
- Cluster operators

## Inputs

- Deployment mode: local demo, standalone chart, or GPU Operator-managed
- Cluster type, Kubernetes version, runtime, Helm version, node architecture, and current context
- GPU model, driver version/root, CDI, NFD/GFD, MIG, VFIO, MNNVL, and IMEX state
- Desired resource family and feature gates
- Chart version, image source, namespace, release name, and existing values
- Approval status for cluster mutation and cleanup

## Workflow

1. Establish the exact cluster context and target. Treat version, prerequisite, feature-gate, and known-issue details as time-sensitive; verify current repository and official product docs.
2. Choose the authoritative flow:
   - GPU Operator installed or planned: follow the current NVIDIA GPU Operator DRA guide.
   - Standalone production-like chart: follow `site/content/docs/prerequisites.md`, `install.md`, `upgrade.md`, and the release-tagged chart docs.
   - Local development: follow scripts under `demo/clusters/kind/` or GPU-aware `demo/clusters/nvkind/`; use provider-specific directories for GKE or OpenShift.
3. Select GPU resources, ComputeDomains, or both. Confirm the matching hardware, feature gates, driver root, CDI, NFD/GFD, IMEX, and chart values.
4. Pin the chart version and review `deployments/helm/dra-driver-nvidia-gpu/values.yaml`, `Chart.yaml`, and `templates/validation.yaml`. Do not copy defaults from a different release.
5. Render before applying:
   - Run `make helm-lint`.
   - Run `helm template` with the exact intended values.
   - Inspect namespaces, images, service accounts, RBAC, host paths, security context, node selectors, feature gates, controller/plugin enablement, and DeviceClasses.
6. Present the mutation plan, affected cluster objects, rollback/cleanup plan, and verification steps. Obtain explicit approval before creating a cluster or running Helm/kubectl mutations.
7. Apply only the approved commands. Keep authentication credentials and Kubernetes client configuration contents out of command lines, shell history, rendered values, manifests, logs, and responses.
8. Verify in order:
   - Helm release and driver pod readiness
   - Expected plugin/controller/webhook containers
   - Expected DeviceClasses
   - `gpu.nvidia.com` and/or `compute-domain.nvidia.com` ResourceSlices
   - ResourceClaim allocation/reservation and pod scheduling
   - Container-visible GPU/CDI devices or IMEX channel/mounts
   - Relevant events and component logs
9. Prefer an existing release-matched manifest under `demo/specs/` or the maintained docs. Select the Kubernetes DRA API version that matches the cluster.
10. Distinguish a local demonstration from a supported GPU Operator deployment. Capture known issues, support status, and hardware limitations from current release sources.
11. Clean up only exact approved objects. Confirm whether workloads, claims, ComputeDomains, namespaces, Helm releases, or demo clusters contain state the operator wants to keep.
12. Return the rendered plan or execution report with versions, values, evidence, deviations, and cleanup state.

## Expected Outputs

- Helm values and rendered-deployment review
- Plan/apply runbook
- Demo or quickstart procedure
- Sample workload manifest and validation report

## References

- Standalone prerequisites: `site/content/docs/prerequisites.md`
- Standalone install and validation: `site/content/docs/install.md`
- Upgrades: `site/content/docs/upgrade.md`
- Chart authority: `deployments/helm/dra-driver-nvidia-gpu/`
- Local clusters: `demo/clusters/`
- Samples: `demo/specs/`
- Public project docs: https://dra-driver-nvidia-gpu.sigs.k8s.io/
- Public documentation navigation history: https://github.com/kubernetes-sigs/dra-driver-nvidia-gpu/issues/1103
- Public GPU Operator DRA guide: https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/dra-intro-install.html
- Public Kubernetes DRA concepts: https://kubernetes.io/docs/concepts/scheduling-eviction/dynamic-resource-allocation/
- Internal supplemental: none provided by this skill

## Validation

- Record Kubernetes, driver, chart, GPU Operator, NVIDIA driver, runtime/CDI, and hardware context.
- Verify the rendered chart before mutation.
- Verify only the expected resource families and DeviceClasses are enabled.
- For the scenario "run the basic GPU sharing sample", verify one claim is allocated, the pod is running, both containers reference the intended claim, and observed GPU UUIDs match the expected sharing result.
- For the scenario "run a ComputeDomain sample", verify clique labels, ComputeDomain/controller objects, daemon readiness evidence, channel claim allocation, and injected channel device plus `/imexd`.
- State whether the result is a local demo, experimental feature, or productized GPU Operator flow.

## Safety Constraints

- Require explicit approval before `helm install/upgrade/uninstall`, mutating `kubectl`, cluster creation/deletion, or invasive tests.
- Show the current context, namespace, release name, and exact target before mutation.
- Never request, display, or reproduce Kubernetes client configuration files, authentication tokens, registry credentials, or private cluster data. Use only minimal redacted excerpts, and omit sensitive values from commands, rendered manifests, logs, and responses.
- Avoid destructive cleanup globs and broad namespace deletion.

## Non-Goals

- Do not present local demo scripts as production support guidance.
- Do not assume current online prerequisites apply to an older release.
- Do not mutate a cluster merely to answer a configuration or explanation question.

## Owners

- DRA engineering
- DRA documentation maintainers
- Cluster operator for each execution

## Refresh Policy

- Review every DRA driver and GPU Operator release.
- Review after Helm value, prerequisite, Kubernetes DRA API, runtime/CDI, demo-script, or known-issue changes.
- Verify official online sources immediately before a productized install or upgrade.

## Registration Notes

Keep this `SKILL.md` as the shared source of truth. Use `agents/openai.yaml` only as registration metadata.
