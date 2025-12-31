# Function Mesh Operator CRD Issue

This repository demonstrates a common installation issue with the **Function Mesh Operator** on Kubernetes when the required Custom Resource Definitions (CRDs) are missing.  

It also provides steps to reproduce the issue and how to resolve it.

---

## **Problem**

When installing `function-mesh-operator` with Helm **without installing CRDs first**, the operator pod may start but will fail with the following error:

error: 
2024-11-20T18:11:25Z ERROR controller-runtime.source.EventHandler if kind is a CRD, it should be installed before calling Start {"kind": "Sink.compute.functionmesh.io", "error": "no matches for kind "Sink" in version "compute.functionmesh.io/v1alpha1""}


**Cause:** The CRDs for `Sink` and other custom resources used by the operator are not installed. Kubernetes cannot recognize these resources until the CRDs are applied.

---

## **Steps to Reproduce**

1. **Install the Function Mesh Operator without CRDs**

helm install function-mesh function-mesh/function-mesh-operator -n operators --set admissionWebhook.enabled=false

2. **Check the logs of the operator pod**
kubectl logs -n operators <function-mesh-operator-pod-name>

3. **Observe errors about missing CRDs, e.g.:**
no matches for kind "Sink" in version "compute.functionmesh.io/v1alpha1"

---
## Resolution

1. **Install the required CRDs**
kubectl apply -f https://functionmesh.io/crds/compute.functionmesh.io_sinks.yaml
# Or apply all CRDs if available:
kubectl apply -f https://functionmesh.io/crds/

2. **Verify CRDs are installed**
kubectl get crds | grep compute.functionmesh.io
- You should see CRDs like Sink.compute.functionmesh.io listed.

3. **Reinstall or upgrade the Function Mesh Operator**
helm upgrade function-mesh function-mesh/function-mesh-operator -n operators --set admissionWebhook.enabled=false

4. **Check logs to ensure no errors**
kubectl logs -n operators <function-mesh-operator-pod-name>

---
## Best Practices
- Always install CRDs before deploying operators that depend on them.
- Use Helm --set admissionWebhook.enabled=false if you don’t want to run cert-manager for the webhook.
- Verify CRDs are applied with kubectl get crds before starting operator pods.
