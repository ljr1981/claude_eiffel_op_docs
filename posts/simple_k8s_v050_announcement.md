# simple_k8s v0.5.0: Production-Ready Kubernetes Orchestration for Eiffel

**90 tests passing. Security-hardened. Ready to deliver Simple Eiffel to the world.**

---

## TL;DR

simple_k8s is now production-ready with comprehensive Kubernetes API coverage, CI/CD integration tools, and security-focused DBC contracts. This library will be the backbone for deploying Simple Eiffel libraries to the world via Kubernetes.

**Key highlights:**
- 90 tests passing (up from 45)
- KUBECTL_QUICK, MANIFEST_BUILDER, K8S_CI_QUICK utilities
- Security tests validating RFC 1123 name compliance
- Full Design by Contract coverage across all 16 classes

---

## The Journey: v0.1 to v0.5

### v0.1 - Foundation (Dec 15)
- K8S_CLIENT core facade
- Kubeconfig parsing (auto-detect ~/.kube/config)
- In-cluster service account detection
- K8S_AUTH with bearer token and client cert support
- K8S_ERROR with HTTP status classification

### v0.2 - Workloads (Dec 16)
- K8S_POD, K8S_DEPLOYMENT, K8S_SERVICE resources
- POD_SPEC, DEPLOYMENT_SPEC, SERVICE_SPEC fluent builders
- Full CRUD operations for pods, deployments, services
- Pod logs, deployment scaling, rolling restarts

### v0.3 - Config Resources (Dec 17)
- K8S_NAMESPACE with phase status
- K8S_CONFIGMAP with data access
- K8S_SECRET with base64 decoding and type detection

### v0.4 - Utilities (Dec 17)
- **KUBECTL_QUICK** - One-liner convenience API
- **MANIFEST_BUILDER** - YAML generation for kubectl apply
- **K8S_CI_QUICK** - CI/CD pipeline operations with exit codes

### v0.5 - Security Hardening (Dec 17)
- 9 security-focused tests
- RFC 1123 name validation (K8s naming rules)
- Path traversal attack prevention
- JSON escaping verification
- DBC contract audit across all classes

---

## What We Have Now

### 16 Production-Ready Classes

| Class | Purpose |
|-------|---------|
| K8S_CLIENT | Main API facade |
| K8S_CONFIG | Kubeconfig parsing |
| K8S_AUTH | Authentication |
| K8S_ERROR | Error handling |
| K8S_POD | Pod resources |
| K8S_DEPLOYMENT | Deployment resources |
| K8S_SERVICE | Service resources |
| K8S_NAMESPACE | Namespace resources |
| K8S_CONFIGMAP | ConfigMap with data access |
| K8S_SECRET | Secret with base64 decoding |
| POD_SPEC | Pod fluent builder |
| DEPLOYMENT_SPEC | Deployment fluent builder |
| SERVICE_SPEC | Service fluent builder |
| KUBECTL_QUICK | One-liner convenience API |
| MANIFEST_BUILDER | YAML manifest generation |
| K8S_CI_QUICK | CI/CD pipeline helper |

### Security Features

```eiffel
-- RFC 1123 name validation
spec.is_valid_k8s_name ("my-pod")       -- True
spec.is_valid_k8s_name ("MyPod")        -- False (uppercase)
spec.is_valid_k8s_name ("../escape")    -- False (path traversal)
spec.is_valid_k8s_name ("my_pod")       -- False (underscore)

-- Names validated against:
-- - Lowercase alphanumeric + hyphens only
-- - Max 253 characters
-- - No starting/ending hyphens
-- - No path traversal sequences
```

### CI/CD Integration

```eiffel
create ci.make
ci.set_namespace ("production")

-- Deploy with verification
if ci.scale_and_wait ("my-app", 3, 60) then
    print ("Scaled to 3 replicas%N")
end

-- Exit codes for CI pipelines
-- 0=success, 1=failure, 2=not_found, 3=timeout, 4=auth_failure, 5=not_ready
```

### YAML Manifest Generation

```eiffel
create builder.make
builder.add_namespace ("production")
builder.add_deployment ("web-app", "nginx:alpine", 3)
builder.add_service ("web-app", 80)
builder.save_to_file ("deployment.yaml")
-- Creates multi-document YAML for kubectl apply -f
```

---

## Where We're Going Next

### Friday (or sooner): First Live Deployment

The immediate goal is using simple_k8s on a local Windows PC to stage our first test of the system to deliver Simple Eiffel to the world.

**The plan:**
1. Local Kubernetes cluster (Docker Desktop or minikube)
2. Deploy Simple Eiffel documentation site
3. Deploy library distribution infrastructure
4. Test the full CI/CD pipeline from Eiffel code

### Future Features

- Watch/streaming operations (real-time pod status)
- Ingress and NetworkPolicy resources
- Helm-like templating integration

---

## Test Coverage

**90 tests across 15 test categories:**

- K8S_ERROR: 6 tests
- K8S_CONFIG: 4 tests
- K8S_AUTH: 4 tests
- K8S_CLIENT: 3 tests
- K8S_NAMESPACE: 4 tests
- K8S_CONFIGMAP: 5 tests
- K8S_SECRET: 7 tests
- K8S_POD: 6 tests
- K8S_DEPLOYMENT: 4 tests
- K8S_SERVICE: 5 tests
- POD_SPEC: 6 tests
- DEPLOYMENT_SPEC: 5 tests
- SERVICE_SPEC: 5 tests
- Edge Cases: 5 tests
- MANIFEST_BUILDER: 8 tests
- K8S_CI_QUICK: 2 tests
- KUBECTL_QUICK: 2 tests
- **Security Tests: 9 tests**

---

## Links

- **Repository**: [github.com/simple-eiffel/simple_k8s](https://github.com/simple-eiffel/simple_k8s)
- **Documentation**: [simple-eiffel.github.io/simple_k8s](https://simple-eiffel.github.io/simple_k8s)
- **Organization**: [github.com/simple-eiffel](https://github.com/simple-eiffel)

---

*Released December 17, 2025*
*Human+AI collaboration: Larry Rix with Claude Opus 4.5 (Anthropic)*
