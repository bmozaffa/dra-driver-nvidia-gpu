---
name: dra-operations-troubleshooting-observability
description: Triage NVIDIA GPU DRA driver installation, scheduling, allocation, prepare/unprepare, CDI, checkpoint, GPU, MIG, VFIO, ComputeDomain, IMEX, upgrade, health, log, event, and Prometheus metric failures. Use when given failed pods or claims, missing devices, broken ResourceSlices, controller or daemon symptoms, upgrade regressions, or a support escalation that needs a bounded evidence plan.
---

# DRA Operations, Troubleshooting, and Observability

## Purpose

Localize an operational symptom to the correct Kubernetes, DRA driver, GPU Operator, runtime, NVIDIA driver, IMEX, or hardware boundary using read-only evidence first.

## Triggers

- Investigate install, chart, pod readiness, scheduling, allocation, injection, or upgrade failures.
- Analyze events, component logs, ResourceClaims, ResourceSlices, checkpoints, or missing devices.
- Inspect metrics, health, latency, errors, or prepared-device state.
- Prepare a support escalation or incident handoff.

## Audience

- Support engineers
- Cluster operators
- DRA engineering and QA

## Inputs

- Symptom, impact, start time, last known good state, and recent changes
- Cluster, Kubernetes, driver/chart, GPU Operator, NVIDIA driver, runtime/CDI, and hardware versions
- Redacted manifests, Helm values, events, object status, logs, and metrics
- Affected nodes, resource family, claims, and workloads
- Approval status for mutation or privileged host inspection

## Workflow

1. Define the scope before collecting data: affected resource family, namespaces, nodes, claims, workloads, time window, versions, deployment mode, and recent upgrade/configuration changes.
2. Protect sensitive data. Ask only for the minimum redacted diagnostic excerpts required; never request or reproduce complete Kubernetes client configuration files, authentication tokens, secrets, private keys, or customer-identifying bundles.
3. Check the installation layer:
   - Helm release/values and enabled components
   - Driver pod/container readiness and restarts
   - Node selectors, tolerations, RBAC, admission, host paths, driver root, and feature-gate validation
4. Check the Kubernetes DRA layer:
   - DeviceClasses and ResourceSlices
   - ResourceClaim/ResourceClaimTemplate allocation and reservation
   - Pod scheduling, node assignment, conditions, and ordered events
5. Check the responsible component:
   - GPU allocation: `gpu-kubelet-plugin`
   - ComputeDomain channel/daemon resources: `compute-domain-kubelet-plugin`
   - ComputeDomain object lifecycle: controller
   - IMEX process/peer behavior: daemon or approved host service
   - Opaque config admission: webhook
6. Check the node/runtime boundary: kubelet prepare/unprepare errors, plugin health, checkpoint state, CDI specs, container runtime CDI support, device files/mounts, and container-visible result.
7. Check the vendor/hardware boundary: NVIDIA driver/NVML, MIG layout, IOMMU/VFIO binding, GPU health, GFD clique labels, `nvidia-imex`, MNNVL fabric, and GPU Operator-managed components.
8. Use logs and events to build a timeline around the first failure. Correlate claim UID, pod, node, driver, operation, and retry; do not dump unrelated logs.
9. Use metrics when enabled in Helm values. Inspect the code-defined `nvidia_dra_*` series, including requests, duration, in-flight operations, prepared devices, prepare/unprepare errors, and ComputeDomain status. Confirm labels and endpoint configuration in the selected release before querying.
10. Classify the failure boundary and test the cheapest falsifiable hypothesis next. Keep mutations separate from diagnostics.
11. Before any recovery action, state blast radius, active claims/workloads, rollback, and evidence-preservation needs. Obtain explicit approval for restart, reinstall, claim/workload deletion, MIG/VFIO/IMEX changes, checkpoint manipulation, upgrade, or cleanup.
12. Escalate with a compact bundle: redacted versions/config, timeline, relevant objects/events/logs/metrics, layer classification, attempted actions, reproducibility, and remaining questions.

## Expected Outputs

- Bounded triage tree
- Root-cause or likely-boundary analysis
- Safe recovery plan
- Redacted support escalation report

## References

- Install and initial validation: `site/content/docs/install.md`
- Prerequisites: `site/content/docs/prerequisites.md`
- Upgrade guidance: `site/content/docs/upgrade.md`
- Architecture: `site/content/docs/concepts/architecture.md`
- GPU and ComputeDomain guides: `site/content/docs/guides/`
- Metrics implementation: `pkg/metrics/`
- Health, checkpoints, and CDI: both kubelet-plugin directories under `cmd/`
- Helm metrics and health configuration: `deployments/helm/dra-driver-nvidia-gpu/values.yaml`
- Public GPU Operator DRA and known issues: https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/dra-intro-install.html
- Public Kubernetes DRA concepts: https://kubernetes.io/docs/concepts/scheduling-eviction/dynamic-resource-allocation/
- Public Kubernetes DRA hardening: https://kubernetes.io/docs/concepts/security/hardening-guide/dynamic-resource-allocation/
- Internal supplemental: none provided by this skill

## Validation

- State the versions, deployment mode, resource family, scope, and evidence window.
- Identify the first failing boundary rather than listing generic checks.
- Tie every conclusion to an object, event, log, metric, code path, or documented prerequisite.
- For the ADR scenario "pod is Pending and no GPU appears", distinguish absent DeviceClass/ResourceSlice, selector mismatch, unallocated claim, scheduler failure, node prepare failure, CDI/runtime failure, and NVIDIA driver/hardware failure.
- For a ComputeDomain incident, include clique labels, controller objects, daemon/host readiness, channel claim, and injection evidence.
- Mark hypotheses, confirmed facts, attempted mutations, and remaining unknowns separately.

## Safety Constraints

- Use read-only diagnostics first.
- Require explicit approval before cluster or host mutation, destructive cleanup, checkpoint removal, service changes, or external support uploads.
- Minimize and redact logs; do not collect secrets or broad customer data.
- Follow Kubernetes security reporting guidance for suspected vulnerabilities rather than public issue discussion.

## Non-Goals

- Do not prescribe a generic restart as root-cause analysis.
- Do not blame the DRA driver before separating scheduler, GPU Operator, runtime, driver, IMEX, and hardware boundaries.
- Do not promise supportability from an unreproduced workaround.

## Owners

- DRA support
- DRA engineering
- DRA QA

## Refresh Policy

- Review every release and after logging, metrics, health, checkpoint, component, chart, prerequisite, or known-issue changes.
- Verify online GPU Operator known issues immediately before a support handoff.
- Refresh escalation fields when support ownership changes.

## Registration Notes

Keep this `SKILL.md` as the shared source of truth. Use `agents/openai.yaml` only as registration metadata.
