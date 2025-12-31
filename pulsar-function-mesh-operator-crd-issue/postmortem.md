# Postmortem: Function Mesh Operator CRD Installation Issue

**Date:** 2024-11-20  
**Severity:** Medium  
**Components:** Function Mesh Operator, Kubernetes, Helm

---

## **Incident Summary**

After installing the `function-mesh-operator` using Helm **without first applying the required CRDs**, the operator pod started but was failing to recognize custom resources such as `Sink`.  

**Error seen:**

no matches for kind "Sink" in version "compute.functionmesh.io/v1alpha1"

---

## **Root Cause**

- The Function Mesh Operator depends on CRDs (`Sink`, `Source`, etc.) defined in `compute.functionmesh.io/v1alpha1`.
- CRDs were not installed prior to the operator deployment.
- Kubernetes requires CRDs to exist before any resources of that kind can be recognized or managed by an operator.

---

## **Impact**

- Operator pods start but fail to manage custom resources.
- Any dependent Function Mesh workloads cannot be created or managed.
- Logs are flooded with CRD-related errors, making troubleshooting harder.

---

## **Resolution**

1. **Install missing CRDs:**
kubectl apply -f https://functionmesh.io/crds/compute.functionmesh.io_sinks.yaml
kubectl apply -f https://functionmesh.io/crds/

2. **Verify CRDs are present:**
kubectl get crds | grep compute.functionmesh.io

3. **Reinstall or upgrade the operator:**
helm upgrade function-mesh function-mesh/function-mesh-operator -n operators --set admissionWebhook.enabled=false

4. **Confirm that logs no longer report missing CRDs.**

---
## Preventative Measures
- Always apply CRDs before deploying operators.
- Include CRD installation as a step in Helm charts or automation scripts.
- Document dependencies (like CRDs) clearly for new cluster setups.
- Monitor operator logs after installation to catch any misconfigurations early.

---
## Lessons Learned
- Operators are dependent on CRDs and will fail silently or log errors if CRDs are missing.
- Admission webhooks (like in function-mesh-operator) can be disabled temporarily for testing without cert-manager.
- Clear documentation of CRD prerequisites is essential for smooth installations.