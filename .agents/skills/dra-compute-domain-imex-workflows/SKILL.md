---
name: dra-compute-domain-imex-workflows
description: Explain, design, review, or troubleshoot NVIDIA DRA ComputeDomain workflows across MNNVL hardware, IMEX daemon lifecycle, driver-managed or host-managed mode, ComputeDomain and ComputeDomainClique resources, per-domain DaemonSets, channel claims, readiness, kubelet prepare, and workload device or mount injection. Use when asked about GB200 or GB300 fabrics, clique state, IMEX channels, or multi-node workload failures.
---

# DRA ComputeDomain and IMEX Workflows

## Purpose

Trace a ComputeDomain from custom-resource creation through IMEX coordination and channel preparation to the exact workload-visible device and mount.

## Triggers

- Explain or create a ComputeDomain and channel-consuming workload.
- Diagnose IMEX daemon, clique, readiness, channel claim, controller, DNS, or MNNVL failures.
- Review driver-managed versus host-managed IMEX behavior.
- Analyze ComputeDomain CRDs, controller reconciliation, daemon lifecycle, or channel injection.

## Audience

- Contributors
- Cluster operators
- Support engineers

## Inputs

- DRA driver/chart, Kubernetes, GPU Operator, and NVIDIA driver versions
- MNNVL system, clique topology/labels, driver root, GFD, and IMEX package/service state
- Helm `resources.computeDomains` values and feature gates
- ComputeDomain, ComputeDomainClique, DaemonSet, ResourceClaimTemplate, ResourceClaim, Pod, event, and log evidence
- Selected IMEX mode, isolation, socket path, and channel allocation mode

## Workflow

1. Verify current behavior from `site/content/docs/concepts/compute-domains.md`, `site/content/docs/guides/compute-domain-workloads.md`, prerequisites, API docs, chart values, and code.
2. Confirm the environment floor and MNNVL facts. Check expected `nvidia.com/gpu.clique` labels, participating nodes, driver/GFD state, and IMEX service ownership.
3. Determine the IMEX lifecycle mode from Helm values and feature gates:
   - `driverManaged`: the driver creates per-ComputeDomain daemon workload and owns its lifecycle.
   - `hostManaged`: the cluster operator owns host `nvidia-imex`; verify the gate, mode, isolation, control socket, `nvidia-imex-ctl`, and readiness contract in the current checkout.
4. Trace the driver-managed control path:
   - A user creates `resource.nvidia.com/v1beta1` `ComputeDomain`.
   - `cmd/compute-domain-controller/` reconciles it, creates the per-domain DaemonSet and channel/daemon claim templates, and maintains related state.
   - `cmd/compute-domain-daemon/` supervises `nvidia-imex` on participating nodes.
   - Daemons publish membership, address, clique, index, and readiness through `ComputeDomainClique` resources when that gate is active.
5. Trace the workload path:
   - The workload references the controller-created channel `ResourceClaimTemplate`.
   - Kubernetes allocates a `compute-domain-default-channel.nvidia.com` device.
   - Kubelet calls `cmd/compute-domain-kubelet-plugin/` prepare.
   - The plugin verifies local IMEX readiness and claim/domain relationship.
   - CDI/runtime setup injects `/dev/nvidia-caps-imex-channels/chan*` and `/imexd` as required.
6. Treat `ComputeDomain.status` as informational unless the current code/docs explicitly define a gating contract. Use claim preparation and local daemon readiness as the workload boundary.
7. Distinguish `Single` and `All` channel allocation modes and verify expected channel count.
8. Diagnose in layers: chart/component enablement → clique labels/topology → controller reconciliation → DaemonSet or host service → ComputeDomainClique/readiness → ResourceSlice/DeviceClass → channel claim allocation → plugin prepare → injected device/mount.
9. Check feature-gate dependencies in code, especially clique/DNS and host-managed behavior. Report code/docs drift rather than guessing.
10. For upgrades or mode changes, protect active ComputeDomains and claims. Treat driver-managed/host-managed transitions and checkpoint changes as explicit migration work.
11. Return a source-backed sequence or triage report with versions, mode, topology, evidence, failure boundary, and safe next checks.

## Expected Outputs

- ComputeDomain/IMEX sequence explanation
- Workload or channel-claim design
- Bounded multi-node triage report
- Mode and upgrade compatibility checklist

## References

- ComputeDomain concepts: `site/content/docs/concepts/compute-domains.md`
- Architecture: `site/content/docs/concepts/architecture.md`
- Workload guide: `site/content/docs/guides/compute-domain-workloads.md`
- Prerequisites: `site/content/docs/prerequisites.md`
- API reference: `site/content/docs/reference/api.md`
- Helm authority: `deployments/helm/dra-driver-nvidia-gpu/`
- Implementation: `cmd/compute-domain-controller/`, `cmd/compute-domain-daemon/`, `cmd/compute-domain-kubelet-plugin/`, `pkg/imex/`
- API types: `api/nvidia.com/resource/v1beta1/`
- Public NVIDIA ComputeDomain background: https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/dra-cds.html
- Public GPU Operator DRA install guide: https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/dra-intro-install.html
- Internal supplemental: none provided by this skill

## Validation

- Confirm mode, feature gates, IMEX service owner, socket/readiness path, isolation, allocation mode, and clique topology.
- Verify the controller-created objects and owner references match the selected mode.
- Verify every expected node or clique has readiness evidence.
- Verify the channel claim is allocated and the plugin prepare path injects the expected channel device and `/imexd`.
- For the ADR scenario, explain ComputeDomain creation → controller reconciliation → IMEX daemon supervision → clique state → channel claim → workload injection.
- Separate driver defects from GFD labels, NVIDIA driver/IMEX package, host service, Kubernetes DRA, runtime, DNS, and hardware fabric issues.

## Safety Constraints

- Use read-only inspection first and obtain explicit approval before applying/deleting ComputeDomains, changing host IMEX service state, switching modes, restarting components, or running MNNVL tests.
- Never expose kubeconfigs, credentials, customer topology, private node addresses, or sensitive logs.
- Avoid mode changes while active ComputeDomain workloads exist unless an approved drain and recovery plan is present.

## Non-Goals

- Do not claim ComputeDomains work on non-MNNVL hardware.
- Do not treat status alone as proof that a workload channel is prepared.
- Do not bypass operator ownership of host-managed IMEX.

## Owners

- DRA ComputeDomain engineering
- DRA support

## Refresh Policy

- Review every DRA driver and GPU Operator release.
- Review after ComputeDomain CRD, clique, DNS, IMEX mode/isolation, driver prerequisite, MNNVL platform, or checkpoint changes.
- Verify current online prerequisites before operator-facing guidance.

## Registration Notes

Keep this `SKILL.md` as the shared source of truth. Use `agents/openai.yaml` only as registration metadata.
