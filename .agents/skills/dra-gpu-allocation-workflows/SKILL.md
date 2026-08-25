---
name: dra-gpu-allocation-workflows
description: Explain, design, review, or troubleshoot NVIDIA GPU DRA allocation across full GPUs, MIG, dynamic MIG, VFIO, time-slicing, MPS, DeviceClasses, ResourceClaims, ResourceSlices, scheduler binding, kubelet prepare/unprepare, checkpoints, and CDI injection. Use when asked about GPU selectors, attributes, capacity, sharing, passthrough, scheduling, allocation failures, or container-visible devices.
---

# DRA GPU Allocation Workflows

## Purpose

Trace a GPU request from declared intent through scheduler allocation and node preparation to the exact device, sharing, and CDI result.

## Triggers

- Explain or create a GPU, MIG, VFIO, time-slicing, or MPS claim.
- Diagnose Pending pods, unallocated claims, prepare/unprepare failures, missing devices, or stale inventory.
- Review DeviceClasses, ResourceSlice attributes/capacity, CEL selectors, opaque configuration, or feature-gate constraints.
- Analyze dynamic MIG lifecycle, checkpoint recovery, or CDI injection.

## Audience

- Contributors
- Cluster operators
- Support engineers

## Inputs

- Driver/chart and Kubernetes versions
- GPU model, MIG/VFIO state, NVIDIA driver, runtime/CDI, and feature gates
- ResourceClaim or ResourceClaimTemplate, Pod, DeviceClass, and ResourceSlice output
- Pod/claim events, gpu-kubelet-plugin logs, checkpoint symptoms, and container evidence
- Whether GPU Operator manages the surrounding stack

## Workflow

1. Verify current behavior from `site/content/docs/concepts/gpu-allocation.md`, the relevant guide/reference pages, and code. Do not extrapolate planned release behavior.
2. Classify the requested resource:
   - Full GPU: `gpu.nvidia.com`
   - MIG slice: `mig.nvidia.com`
   - VFIO passthrough: `vfio.gpu.nvidia.com`
3. Identify the sharing/config contract from `api/nvidia.com/resource/v1beta1/`:
   - `GpuConfig`
   - `MigDeviceConfig`
   - `VfioDeviceConfig`
   - Sharing and IOMMU settings
4. Validate required feature gates and every dependency or mutual exclusion against `pkg/featuregates/featuregates.go` and current docs. Treat code/docs differences as drift to report.
5. Inspect the matching DeviceClass templates and the node's `gpu.nvidia.com` ResourceSlice. Distinguish serialized bare attribute keys from CEL access through `device.attributes['gpu.nvidia.com']` and capacity through `device.capacity['gpu.nvidia.com']`.
6. Trace the allocation path:
   - Pod references a ResourceClaim or template.
   - The scheduler matches DeviceClass, CEL selectors, attributes, and capacity against ResourceSlices and records the allocation/node.
   - Kubelet calls the GPU plugin's prepare path.
   - The plugin validates config and device state, performs any MIG/VFIO/sharing setup, persists prepared state, writes CDI specs, and returns CDI device names.
   - The runtime consumes CDI and starts the container.
   - Unprepare reverses driver-owned state when the claim is released.
7. For static MIG, confirm partitions existed before plugin discovery and restart the plugin after approved out-of-band hardware changes. For dynamic MIG, verify advertised partitionable capacity, Kubernetes gate/version requirements, on-demand creation, teardown, and checkpoint recovery.
8. For VFIO, verify IOMMU, driver binding, feature gates, workload/runtime expectations, and processes that may hold the GPU. Treat host driver changes as privileged operations.
9. For sharing, distinguish time-slicing, MPS, MIG hardware isolation, and plain multi-container use of the same claim. Do not describe them as equivalent isolation models.
10. Diagnose in layers: DeviceClass → ResourceSlice inventory/health → claim allocation → scheduler events → node assignment → plugin prepare → checkpoint/CDI → runtime/container.
11. Use maintained samples under `demo/specs/` and docs under `site/content/docs/guides/gpu-allocation/`; adapt API versions to the target Kubernetes release.
12. Return a request-to-injection explanation or bounded triage report with prerequisites, observed evidence, unsupported assumptions, and validation steps.

## Expected Outputs

- GPU claim or selector design
- Request-to-CDI flow explanation
- Allocation triage report
- Compatibility and validation checklist

## References

- GPU concepts: `site/content/docs/concepts/gpu-allocation.md`
- API types: `site/content/docs/reference/api.md`, `api/nvidia.com/resource/v1beta1/`
- ResourceSlice fields and CEL names: `site/content/docs/reference/resourceslice-attributes.md`
- Feature gates: `site/content/docs/reference/feature-gates.md`, `pkg/featuregates/featuregates.go`
- Task guides: `site/content/docs/guides/gpu-allocation/`
- Implementation: `cmd/gpu-kubelet-plugin/`
- Samples and tests: `demo/specs/`, `test/e2e/`, `tests/bats/`
- Public Kubernetes DRA concepts: https://kubernetes.io/docs/concepts/scheduling-eviction/dynamic-resource-allocation/
- Public GPU Operator DRA guide: https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/dra-intro-install.html
- Internal supplemental: none provided by this skill

## Validation

- Identify DeviceClass, requested device count, selectors, opaque config, feature gates, and expected sharing/isolation semantics.
- Confirm the allocated device and node from claim status and ResourceSlice data.
- Confirm prepare succeeded and a CDI device reached the intended container.
- For the ADR scenario, explain ResourceClaim → scheduler binding → kubelet prepare → CDI injection and name the evidence available at each boundary.
- For dynamic MIG, verify actual partition creation and later teardown, not only scheduler allocation.
- For failures, separate driver behavior from Kubernetes scheduler, GPU Operator, NVIDIA driver, runtime/CDI, and hardware configuration.

## Safety Constraints

- Use read-only cluster inspection first and obtain explicit approval before applying claims, restarting plugins, changing MIG/VFIO state, editing host drivers, deleting checkpoints, or running invasive tests.
- Use repository content, public documentation, user-provided material, and approved test fixtures. Keep private operational data out of commands, logs, patches, generated artifacts, and responses.
- Preserve active workloads and claims when suggesting recovery.

## Non-Goals

- Do not claim unsupported hardware, feature gates, or releases are production-ready.
- Do not confuse DRA with the legacy extended-resource device-plugin path.
- Do not treat time-slicing or MPS as MIG-equivalent hardware isolation.

## Owners

- DRA GPU engineering
- DRA support

## Refresh Policy

- Review every DRA driver release and Kubernetes minor bump.
- Review after GPU API, ResourceSlice attribute, feature-gate, MIG, VFIO, checkpoint, CDI, or GPU Operator changes.
- Verify current public prerequisites and known issues before operator-facing guidance.
