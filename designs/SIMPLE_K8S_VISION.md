# simple_k8s Vision Document

**Kubernetes Cluster Orchestration for Eiffel Applications**

*Research Date: December 17, 2025*
*Status: VISION (Pre-Development)*

---

## Executive Summary

`simple_k8s` is a proposed Eiffel library for programmatic Kubernetes cluster management. It would enable Eiffel applications to deploy, scale, and manage containerized workloads on Kubernetes clusters, building on top of `simple_docker` to bring full container orchestration capabilities to the Simple Eiffel ecosystem.

**Primary Goal**: Enable Eiffel developers to deploy and manage Kubernetes workloads programmatically with Design by Contract guarantees.

**Relationship to simple_docker**: While `simple_docker` manages individual containers on a single host, `simple_k8s` orchestrates containers across clusters with scaling, self-healing, and service discovery.

---

## Research Findings

### 1. Existing Kubernetes SDKs

| Language | SDK | Status | Notes |
|----------|-----|--------|-------|
| **Go** | client-go (official) | Production | Most complete, reference implementation |
| **Python** | kubernetes-client/python | Production | Official, widely used |
| **Java** | kubernetes-client/java | Production | Official |
| **C** | kubernetes-client/c | **Beta** | Official, v0.14.0 (Nov 2025) |
| **Rust** | kube-rs | Community | Popular, async-first |
| **.NET** | KubernetesClient | Community | Official-ish |

**Key Insight**: The Kubernetes C client is official but Beta. It wraps the REST API and provides comprehensive coverage of Kubernetes resources. We can either wrap it or implement direct REST API calls.

### 2. Kubernetes API Architecture

The Kubernetes API is a RESTful interface over HTTP/HTTPS:

| API Group | REST Path | Resources |
|-----------|-----------|-----------|
| **Core (v1)** | `/api/v1` | Pods, Services, ConfigMaps, Secrets, Namespaces, Nodes |
| **apps/v1** | `/apis/apps/v1` | Deployments, StatefulSets, DaemonSets, ReplicaSets |
| **batch/v1** | `/apis/batch/v1` | Jobs, CronJobs |
| **networking.k8s.io/v1** | `/apis/networking.k8s.io/v1` | Ingress, NetworkPolicy |
| **rbac.authorization.k8s.io/v1** | `/apis/rbac.authorization.k8s.io/v1` | Roles, RoleBindings, ClusterRoles |
| **storage.k8s.io/v1** | `/apis/storage.k8s.io/v1` | StorageClasses, PersistentVolumes |

**API Versioning**:
- **Alpha** (v1alpha1): Experimental, disabled by default
- **Beta** (v1beta1): Mostly stable, enabled by default
- **Stable** (v1): Long-term support guaranteed

### 3. Core Resource Operations

| HTTP Verb | K8s Verb | Operation |
|-----------|----------|-----------|
| GET | get | Retrieve single resource |
| GET (collection) | list | Retrieve all resources |
| GET (?watch=true) | watch | Stream changes |
| POST | create | Create new resource |
| PUT | update | Replace entire resource |
| PATCH | patch | Partial update |
| DELETE | delete | Remove resource |

### 4. Authentication Methods

| Method | Description | Use Case |
|--------|-------------|----------|
| **kubeconfig** | Config file at `~/.kube/config` | Local development |
| **Service Account** | In-cluster token at `/var/run/secrets/kubernetes.io/serviceaccount/token` | Pods accessing API |
| **Bearer Token** | Long-lived token | CI/CD, automation |
| **Client Certificate** | Mutual TLS | High security environments |
| **OIDC** | OpenID Connect | Enterprise SSO |

**Default Paths**:
- kubeconfig: `$HOME/.kube/config` (all platforms)
- In-cluster CA: `/var/run/secrets/kubernetes.io/serviceaccount/ca.crt`
- In-cluster Token: `/var/run/secrets/kubernetes.io/serviceaccount/token`

### 5. Existing Eiffel Kubernetes Integration

**Current State**: None. No Eiffel library exists for Kubernetes.

**What's Missing**: Everything - this would be the first Eiffel Kubernetes SDK.

**Prerequisite**: `simple_docker` exists (v1.4.0) and handles container basics. `simple_k8s` builds on this foundation.

### 6. Kubernetes C Client Details

From [kubernetes-client/c](https://github.com/kubernetes-client/c):

| Aspect | Detail |
|--------|--------|
| **Version** | v0.14.0 (November 2025) |
| **Status** | Beta, Official |
| **Stars** | 180+ |
| **Dependencies** | libssl, libcurl, libwebsockets, libyaml |
| **Build** | CMake |
| **Auth** | kubeconfig, in-cluster |

**API Pattern**:
```c
// List pods in namespace
list_t *pod_list = CoreV1API_listNamespacedPod(
    apiClient,
    "default",    // namespace
    NULL,         // pretty
    NULL,         // allowWatchBookmarks
    NULL,         // continue
    NULL,         // fieldSelector
    NULL,         // labelSelector
    NULL,         // limit
    NULL,         // resourceVersion
    NULL,         // resourceVersionMatch
    NULL,         // sendInitialEvents
    NULL,         // timeoutSeconds
    NULL          // watch
);
```

**Considerations**:
- C API is verbose (many NULL parameters)
- Requires external dependencies (libcurl, libyaml, libwebsockets)
- Generated code, not hand-crafted
- Thread safety requires explicit setup

---

## Vision: What simple_k8s Should Provide

### Core Capabilities

```
simple_k8s/
├── K8S_CLIENT               -- Main facade (cluster connection + config)
├── K8S_POD                  -- Pod resource with lifecycle
├── K8S_DEPLOYMENT           -- Deployment with scaling/rollout
├── K8S_SERVICE              -- Service for pod discovery
├── K8S_NAMESPACE            -- Namespace isolation
├── K8S_CONFIGMAP            -- Configuration data
├── K8S_SECRET               -- Sensitive data
├── K8S_INGRESS              -- HTTP routing
├── POD_SPEC                 -- Pod configuration with DBC
├── DEPLOYMENT_SPEC          -- Deployment configuration
├── SERVICE_SPEC             -- Service configuration
├── K8S_CONFIG               -- kubeconfig parser
├── K8S_AUTH                 -- Authentication strategies
├── K8S_ERROR                -- Error handling
├── MANIFEST_BUILDER         -- YAML manifest generation
└── KUBECTL_QUICK            -- One-liner convenience API
```

### Phase 1: Core Functionality (MVP)

```eiffel
class K8S_CLIENT

feature -- Connection

    make_with_kubeconfig
            -- Connect using default kubeconfig (~/.kube/config).
        ensure
            connected: is_connected
        end

    make_with_kubeconfig_path (a_path: STRING)
            -- Connect using kubeconfig at `a_path'.
        require
            path_exists: file_exists (a_path)
        ensure
            connected: is_connected
        end

    make_in_cluster
            -- Connect from within a pod using service account.
        require
            running_in_pod: is_in_cluster_environment
        ensure
            connected: is_connected
        end

feature -- Cluster Info

    server_version: STRING
            -- Kubernetes server version.
        require
            connected: is_connected
        end

    namespaces: ARRAYED_LIST [K8S_NAMESPACE]
            -- All namespaces in cluster.
        require
            connected: is_connected
        end

    current_namespace: STRING
            -- Current namespace from kubeconfig context.

feature -- Pod Operations

    list_pods (a_namespace: STRING): ARRAYED_LIST [K8S_POD]
            -- List all pods in `a_namespace'.
        require
            connected: is_connected
            namespace_not_empty: not a_namespace.is_empty
        end

    get_pod (a_name, a_namespace: STRING): detachable K8S_POD
            -- Get pod by name.
        require
            connected: is_connected
            name_not_empty: not a_name.is_empty
        end

    create_pod (a_spec: POD_SPEC; a_namespace: STRING): K8S_POD
            -- Create pod from specification.
        require
            connected: is_connected
            spec_valid: a_spec.is_valid
        ensure
            pod_created: Result /= Void implies Result.exists
        end

    delete_pod (a_name, a_namespace: STRING)
            -- Delete pod.
        require
            connected: is_connected
            name_not_empty: not a_name.is_empty
        end

    pod_logs (a_name, a_namespace: STRING): STRING
            -- Get pod logs (default container).
        require
            connected: is_connected
            name_not_empty: not a_name.is_empty
        end

feature -- Deployment Operations

    list_deployments (a_namespace: STRING): ARRAYED_LIST [K8S_DEPLOYMENT]
            -- List all deployments.
        require
            connected: is_connected
        end

    create_deployment (a_spec: DEPLOYMENT_SPEC; a_namespace: STRING): K8S_DEPLOYMENT
            -- Create deployment from specification.
        require
            connected: is_connected
            spec_valid: a_spec.is_valid
        ensure
            deployment_created: Result /= Void implies Result.exists
        end

    scale_deployment (a_name, a_namespace: STRING; a_replicas: INTEGER)
            -- Scale deployment to `a_replicas'.
        require
            connected: is_connected
            replicas_non_negative: a_replicas >= 0
        end

    rollout_restart (a_name, a_namespace: STRING)
            -- Trigger rolling restart of deployment.
        require
            connected: is_connected
        end

feature -- Service Operations

    list_services (a_namespace: STRING): ARRAYED_LIST [K8S_SERVICE]
            -- List all services.
        require
            connected: is_connected
        end

    create_service (a_spec: SERVICE_SPEC; a_namespace: STRING): K8S_SERVICE
            -- Create service from specification.
        require
            connected: is_connected
            spec_valid: a_spec.is_valid
        end

end
```

### Phase 2: Resource Specifications with DBC

```eiffel
class POD_SPEC

feature -- Configuration

    name: STRING
    namespace: STRING
    image: STRING
    command: detachable ARRAY [STRING]
    args: detachable ARRAY [STRING]
    environment: HASH_TABLE [STRING, STRING]
    ports: ARRAYED_LIST [TUPLE [name: STRING; port: INTEGER]]
    volumes: ARRAYED_LIST [TUPLE [name, mount_path: STRING]]
    labels: HASH_TABLE [STRING, STRING]
    annotations: HASH_TABLE [STRING, STRING]

    -- Resource limits
    cpu_request: detachable STRING      -- e.g., "100m"
    cpu_limit: detachable STRING        -- e.g., "500m"
    memory_request: detachable STRING   -- e.g., "128Mi"
    memory_limit: detachable STRING     -- e.g., "512Mi"

feature -- Builder Pattern

    set_name (a_name: STRING): like Current
        require
            name_valid: is_valid_k8s_name (a_name)
        do
            name := a_name
            Result := Current
        end

    set_image (a_image: STRING): like Current
        require
            image_not_empty: not a_image.is_empty
        do
            image := a_image
            Result := Current
        end

    add_env (a_name, a_value: STRING): like Current
        require
            name_not_empty: not a_name.is_empty
        do
            environment.put (a_value, a_name)
            Result := Current
        end

    add_port (a_name: STRING; a_port: INTEGER): like Current
        require
            valid_port: a_port > 0 and a_port <= 65535
        do
            ports.extend ([a_name, a_port])
            Result := Current
        end

    set_resources (a_cpu_req, a_cpu_lim, a_mem_req, a_mem_lim: STRING): like Current
        do
            cpu_request := a_cpu_req
            cpu_limit := a_cpu_lim
            memory_request := a_mem_req
            memory_limit := a_mem_lim
            Result := Current
        end

feature -- Validation

    is_valid: BOOLEAN
            -- Is specification valid for submission?
        do
            Result := not name.is_empty and not image.is_empty
        end

    is_valid_k8s_name (a_name: STRING): BOOLEAN
            -- Is `a_name' a valid Kubernetes resource name?
            -- Must be lowercase, alphanumeric, -, max 253 chars
        do
            Result := not a_name.is_empty and
                      a_name.count <= 253 and
                      a_name.is_valid_as_string_8 and
                      -- Additional regex validation
                      True
        end

invariant
    name_not_void: name /= Void
    image_not_void: image /= Void

end
```

### Phase 3: Deployment Specification

```eiffel
class DEPLOYMENT_SPEC

feature -- Configuration

    name: STRING
    namespace: STRING
    replicas: INTEGER
    pod_spec: POD_SPEC
    selector: HASH_TABLE [STRING, STRING]
    strategy: STRING  -- "RollingUpdate" or "Recreate"

    -- Rolling update config
    max_unavailable: detachable STRING  -- "25%" or "1"
    max_surge: detachable STRING        -- "25%" or "1"

feature -- Builder Pattern

    set_name (a_name: STRING): like Current
        do
            name := a_name
            Result := Current
        end

    set_replicas (a_count: INTEGER): like Current
        require
            non_negative: a_count >= 0
        do
            replicas := a_count
            Result := Current
        end

    set_pod_spec (a_spec: POD_SPEC): like Current
        require
            spec_valid: a_spec.is_valid
        do
            pod_spec := a_spec
            Result := Current
        end

    set_rolling_update (a_max_unavailable, a_max_surge: STRING): like Current
        do
            strategy := "RollingUpdate"
            max_unavailable := a_max_unavailable
            max_surge := a_max_surge
            Result := Current
        end

feature -- Validation

    is_valid: BOOLEAN
        do
            Result := not name.is_empty and
                      replicas >= 0 and
                      pod_spec /= Void and then pod_spec.is_valid
        end

end
```

### Phase 4: Service Specification

```eiffel
class SERVICE_SPEC

feature -- Configuration

    name: STRING
    namespace: STRING
    service_type: STRING  -- "ClusterIP", "NodePort", "LoadBalancer"
    selector: HASH_TABLE [STRING, STRING]
    ports: ARRAYED_LIST [SERVICE_PORT]

feature -- Builder Pattern

    set_name (a_name: STRING): like Current
        do
            name := a_name
            Result := Current
        end

    set_cluster_ip: like Current
        do
            service_type := "ClusterIP"
            Result := Current
        end

    set_node_port: like Current
        do
            service_type := "NodePort"
            Result := Current
        end

    set_load_balancer: like Current
        do
            service_type := "LoadBalancer"
            Result := Current
        end

    add_selector (a_key, a_value: STRING): like Current
        do
            selector.put (a_value, a_key)
            Result := Current
        end

    add_port (a_port: SERVICE_PORT): like Current
        do
            ports.extend (a_port)
            Result := Current
        end

end
```

### Phase 5: YAML Manifest Builder

```eiffel
class MANIFEST_BUILDER

feature -- Pod Manifest

    pod_manifest (a_spec: POD_SPEC): STRING
            -- Generate Pod YAML manifest.
        require
            spec_valid: a_spec.is_valid
        do
            Result := yaml_builder
                .add ("apiVersion", "v1")
                .add ("kind", "Pod")
                .start_map ("metadata")
                    .add ("name", a_spec.name)
                    .add ("namespace", a_spec.namespace)
                .end_map
                .start_map ("spec")
                    .start_array ("containers")
                        .start_map
                            .add ("name", a_spec.name)
                            .add ("image", a_spec.image)
                        .end_map
                    .end_array
                .end_map
                .to_yaml
        end

feature -- Deployment Manifest

    deployment_manifest (a_spec: DEPLOYMENT_SPEC): STRING
            -- Generate Deployment YAML manifest.
        require
            spec_valid: a_spec.is_valid
        end

feature -- Service Manifest

    service_manifest (a_spec: SERVICE_SPEC): STRING
            -- Generate Service YAML manifest.
        require
            spec_valid: a_spec.is_valid
        end

feature -- Apply

    apply_manifest (a_yaml: STRING): BOOLEAN
            -- Apply YAML manifest to cluster (like kubectl apply -f).
        require
            yaml_not_empty: not a_yaml.is_empty
        end

end
```

### Phase 6: KUBECTL_QUICK (One-Liner API)

```eiffel
class KUBECTL_QUICK

feature -- Quick Operations

    run (a_name, a_image: STRING): K8S_POD
            -- Quick pod creation (like `kubectl run`).
            -- Example: kubectl.run ("nginx", "nginx:alpine")
        do
            Result := client.create_pod (
                create {POD_SPEC}.make.set_name (a_name).set_image (a_image),
                current_namespace
            )
        end

    expose (a_name: STRING; a_port: INTEGER): K8S_SERVICE
            -- Expose pod as service (like `kubectl expose`).
        do
            Result := client.create_service (
                create {SERVICE_SPEC}.make
                    .set_name (a_name)
                    .add_selector ("app", a_name)
                    .add_port (create {SERVICE_PORT}.make (a_port, a_port)),
                current_namespace
            )
        end

    scale (a_deployment: STRING; a_replicas: INTEGER)
            -- Scale deployment (like `kubectl scale`).
        do
            client.scale_deployment (a_deployment, current_namespace, a_replicas)
        end

    logs (a_pod: STRING): STRING
            -- Get pod logs (like `kubectl logs`).
        do
            Result := client.pod_logs (a_pod, current_namespace)
        end

    exec (a_pod, a_command: STRING): STRING
            -- Execute command in pod (like `kubectl exec`).
        do
            Result := client.exec_in_pod (a_pod, current_namespace, a_command)
        end

    delete (a_resource_type, a_name: STRING)
            -- Delete resource (like `kubectl delete`).
        do
            inspect a_resource_type
            when "pod" then
                client.delete_pod (a_name, current_namespace)
            when "deployment" then
                client.delete_deployment (a_name, current_namespace)
            when "service" then
                client.delete_service (a_name, current_namespace)
            end
        end

end
```

---

## Implementation Approach

### Option A: Wrap Kubernetes C Client

```eiffel
feature {NONE} -- External (C library)

    c_load_kube_config: POINTER
        external "C inline use <kubernetes/config/kube_config.h>"
        alias "return load_kube_config(NULL);"
        end

    c_list_namespaced_pods (a_client, a_namespace: POINTER): POINTER
        external "C inline use <kubernetes/api/CoreV1API.h>"
        alias "return CoreV1API_listNamespacedPod($a_client, $a_namespace, ...);"
        end
```

**Pros**: Reuse official C client, extensive API coverage
**Cons**: Heavy dependencies (libcurl, libyaml, libwebsockets), complex build, Beta status

### Option B: Direct REST API via HTTPS

```eiffel
feature -- Implementation

    list_pods (a_namespace: STRING): ARRAYED_LIST [K8S_POD]
        local
            l_url: STRING
            l_response: STRING
            l_json: JSON_VALUE
        do
            l_url := api_server + "/api/v1/namespaces/" + a_namespace + "/pods"
            l_response := http.get (l_url, auth_headers)
            l_json := json_parser.parse (l_response)
            Result := parse_pod_list (l_json)
        end
```

**Pros**: No external dependencies (use simple_http + simple_json), full control, lighter weight
**Cons**: More implementation work, must handle all edge cases

### Option C: Hybrid (REST + Generated Specs)

Use REST API for communication but auto-generate spec classes from OpenAPI schema.

**Pros**: Best of both worlds
**Cons**: Requires OpenAPI tooling

### Recommendation: Option B (Direct REST API)

The Kubernetes REST API is well-documented and stable. We already have:
- `simple_http` for HTTPS requests
- `simple_json` for JSON parsing
- `simple_yaml` for YAML parsing (kubeconfig)

This approach:
1. Keeps the library self-contained within Simple Eiffel ecosystem
2. Avoids heavy C dependencies (libcurl, libyaml, libwebsockets already in C client)
3. Provides full control over the implementation
4. Matches the pattern established by `simple_docker`

---

## Dependencies

| Dependency | Purpose | Status |
|------------|---------|--------|
| **simple_http** | HTTPS requests to API server | EXISTS |
| **simple_json** | JSON parsing/generation | EXISTS |
| **simple_yaml** | kubeconfig parsing | EXISTS |
| **simple_file** | File operations | EXISTS |
| **simple_docker** | Container reference (optional) | EXISTS (v1.4.0) |

**No new libraries required** - all prerequisites exist.

---

## Development Phases

| Phase | Scope | Effort | Priority |
|-------|-------|--------|----------|
| **1** | K8S_CLIENT connection + auth, Pod CRUD, basic queries | MEDIUM | HIGH |
| **2** | POD_SPEC, DEPLOYMENT_SPEC, SERVICE_SPEC with DBC | MEDIUM | HIGH |
| **3** | Deployment operations (scale, rollout, restart) | MEDIUM | HIGH |
| **4** | Service operations, ConfigMap, Secret | MEDIUM | MEDIUM |
| **5** | MANIFEST_BUILDER (YAML generation) | LOW | MEDIUM |
| **6** | KUBECTL_QUICK convenience API | LOW | MEDIUM |
| **7** | Watch/streaming for real-time updates | HIGH | LOW |
| **8** | Advanced (Ingress, NetworkPolicy, RBAC) | HIGH | LOW |

---

## Actual API Request/Response Examples

### GET /api/v1/namespaces/default/pods

**Request**:
```
GET /api/v1/namespaces/default/pods HTTP/1.1
Host: kubernetes.default.svc
Authorization: Bearer <token>
Accept: application/json
```

**Response**:
```json
{
  "kind": "PodList",
  "apiVersion": "v1",
  "metadata": {
    "resourceVersion": "12345"
  },
  "items": [
    {
      "metadata": {
        "name": "nginx-7d6877d777-abc12",
        "namespace": "default",
        "labels": {
          "app": "nginx"
        }
      },
      "spec": {
        "containers": [
          {
            "name": "nginx",
            "image": "nginx:alpine",
            "ports": [{"containerPort": 80}]
          }
        ]
      },
      "status": {
        "phase": "Running",
        "podIP": "10.244.0.5"
      }
    }
  ]
}
```

### POST /api/v1/namespaces/default/pods

**Request**:
```
POST /api/v1/namespaces/default/pods HTTP/1.1
Host: kubernetes.default.svc
Authorization: Bearer <token>
Content-Type: application/json

{
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": {
    "name": "my-pod",
    "labels": {"app": "myapp"}
  },
  "spec": {
    "containers": [{
      "name": "myapp",
      "image": "myimage:latest",
      "ports": [{"containerPort": 8080}]
    }]
  }
}
```

**Response**:
```json
{
  "kind": "Pod",
  "apiVersion": "v1",
  "metadata": {
    "name": "my-pod",
    "namespace": "default",
    "uid": "12345-67890-abcde",
    "resourceVersion": "54321",
    "creationTimestamp": "2025-12-17T10:00:00Z"
  },
  "spec": { ... },
  "status": {
    "phase": "Pending"
  }
}
```

### GET /apis/apps/v1/namespaces/default/deployments

**Response**:
```json
{
  "kind": "DeploymentList",
  "apiVersion": "apps/v1",
  "items": [
    {
      "metadata": {
        "name": "nginx-deployment",
        "namespace": "default"
      },
      "spec": {
        "replicas": 3,
        "selector": {
          "matchLabels": {"app": "nginx"}
        },
        "template": {
          "metadata": {"labels": {"app": "nginx"}},
          "spec": {
            "containers": [{
              "name": "nginx",
              "image": "nginx:1.21"
            }]
          }
        }
      },
      "status": {
        "replicas": 3,
        "readyReplicas": 3,
        "availableReplicas": 3
      }
    }
  ]
}
```

---

## kubeconfig File Format

```yaml
apiVersion: v1
kind: Config
current-context: my-cluster
clusters:
- name: my-cluster
  cluster:
    server: https://192.168.1.100:6443
    certificate-authority-data: LS0tLS1CRUdJTi...
contexts:
- name: my-cluster
  context:
    cluster: my-cluster
    user: admin
    namespace: default
users:
- name: admin
  user:
    client-certificate-data: LS0tLS1CRUdJTi...
    client-key-data: LS0tLS1CRUdJTi...
```

**K8S_CONFIG class must parse**:
- `clusters[].cluster.server` - API server URL
- `clusters[].cluster.certificate-authority-data` - CA cert (base64)
- `users[].user.client-certificate-data` - Client cert (base64)
- `users[].user.client-key-data` - Client key (base64)
- `users[].user.token` - Bearer token (alternative)
- `contexts[].context.namespace` - Default namespace
- `current-context` - Active context name

---

## Developer Pain Points (Research)

### 1. Kubernetes Operational Challenges (2024)

| Pain Point | Finding | simple_k8s Mitigation |
|------------|---------|----------------------|
| **Complexity** | 75% report operational struggles | Simple facade API with sensible defaults |
| **Troubleshooting** | Multi-component debugging is hard | Detailed error messages with K8S_ERROR |
| **Multi-cluster** | 56% have 10+ clusters | K8S_CLIENT.make_with_context for switching |
| **Developer experience** | 82% struggle with tailored clusters | KUBECTL_QUICK for common operations |
| **Manual tasks** | Too much time on patching/troubleshooting | DBC contracts catch errors early |

### 2. SDK Anti-Patterns to Avoid

| Anti-Pattern | Problem | simple_k8s Approach |
|--------------|---------|---------------------|
| **Verbose APIs** | C client requires many NULL params | Fluent builder pattern with defaults |
| **Stringly-typed** | Errors only at runtime | Strong typing with spec classes + DBC |
| **Hidden defaults** | Unexpected behavior | Explicit configuration, no surprises |
| **Version coupling** | Break on K8s upgrades | API version negotiation |
| **No retries** | Transient failures crash | Retry logic with exponential backoff |

### 3. Key Design Decisions from Pain Points

1. **Fluent builders**: POD_SPEC, DEPLOYMENT_SPEC use builder pattern
2. **Sensible defaults**: Most parameters optional with reasonable defaults
3. **DBC validation**: Preconditions catch errors before API calls
4. **Clear errors**: K8S_ERROR with error code, message, and resource info
5. **Version resilience**: Handle API deprecations gracefully
6. **Retry logic**: Automatic retries for transient failures

---

## Use Cases

### 1. Deploy Eiffel Application

```eiffel
-- Deploy simple_web application to Kubernetes
create k8s.make_with_kubeconfig

-- Create deployment
create dep_spec.make
    .set_name ("my-web-app")
    .set_replicas (3)
    .set_pod_spec (
        create {POD_SPEC}.make
            .set_image ("my-registry/my-web-app:1.0")
            .add_port ("http", 8080)
            .set_resources ("100m", "500m", "128Mi", "512Mi")
    )

deployment := k8s.create_deployment (dep_spec, "production")

-- Expose as service
create svc_spec.make
    .set_name ("my-web-app")
    .set_load_balancer
    .add_selector ("app", "my-web-app")
    .add_port (create {SERVICE_PORT}.make (80, 8080))

service := k8s.create_service (svc_spec, "production")
print ("Service available at: " + service.external_ip)
```

### 2. Scale Based on Load

```eiffel
-- Auto-scale deployment based on metrics
deployment := k8s.get_deployment ("my-web-app", "production")
if deployment.cpu_utilization > 80 then
    k8s.scale_deployment ("my-web-app", "production", deployment.replicas + 2)
    print ("Scaled to " + (deployment.replicas + 2).out + " replicas")
end
```

### 3. Rolling Update

```eiffel
-- Update to new image version
k8s.set_deployment_image ("my-web-app", "production", "my-registry/my-web-app:2.0")

-- Watch rollout status
across k8s.watch_deployment_rollout ("my-web-app", "production") as event loop
    print (event.message)
    if event.is_complete then
        print ("Rollout complete!")
    end
end
```

### 4. CI/CD Integration (with simple_ci)

```eiffel
-- In CI pipeline after Docker build
pipeline.add_stage ("deploy", agent
    local
        k8s: K8S_CLIENT
        kubectl: KUBECTL_QUICK
    do
        create k8s.make_with_kubeconfig_path ("/secrets/kubeconfig")
        create kubectl.make (k8s)

        -- Update deployment image
        k8s.set_deployment_image ("app", "staging", built_image_tag)

        -- Wait for rollout
        k8s.wait_for_rollout ("app", "staging", 300)  -- 5 min timeout

        -- Run smoke tests
        pod := kubectl.run ("smoke-test", "my-test-image:latest")
        k8s.wait_for_pod_completion (pod.name, "staging", 120)

        if pod.exit_code = 0 then
            print ("Deployment successful!")
        else
            k8s.rollout_undo ("app", "staging")
            fail ("Smoke tests failed, rolled back")
        end
    end
)
```

---

## Success Metrics

| Metric | Target |
|--------|--------|
| kubeconfig parsing | Full support (clusters, contexts, users) |
| Pod operations | 100% CRUD coverage |
| Deployment operations | Create, scale, rollout, restart, delete |
| Service operations | ClusterIP, NodePort, LoadBalancer |
| Authentication | kubeconfig, bearer token, in-cluster |
| Error handling | Detailed K8S_ERROR with retry guidance |
| Test coverage | 40+ tests |
| Documentation | IUARC 5-document standard |

---

## Implementation Plan

### Week 1: Foundation

| Task | Files |
|------|-------|
| Create directory structure | simple_k8s/* |
| K8S_CONFIG (kubeconfig parser) | src/k8s_config.e |
| K8S_AUTH (auth strategies) | src/k8s_auth.e |
| K8S_CLIENT connection | src/k8s_client.e |
| K8S_ERROR | src/k8s_error.e |

### Week 2: Core Resources

| Task | Files |
|------|-------|
| POD_SPEC with DBC | src/pod_spec.e |
| K8S_POD | src/k8s_pod.e |
| Pod CRUD operations | src/k8s_client.e |
| Basic tests | testing/lib_tests.e |

### Week 3: Deployments & Services

| Task | Files |
|------|-------|
| DEPLOYMENT_SPEC | src/deployment_spec.e |
| K8S_DEPLOYMENT | src/k8s_deployment.e |
| SERVICE_SPEC | src/service_spec.e |
| K8S_SERVICE | src/k8s_service.e |
| Scale, rollout operations | src/k8s_client.e |

### Week 4: Polish

| Task | Files |
|------|-------|
| KUBECTL_QUICK | src/kubectl_quick.e |
| MANIFEST_BUILDER | src/manifest_builder.e |
| ConfigMap, Secret | src/k8s_configmap.e, k8s_secret.e |
| Documentation | docs/*.html |
| README, CHANGELOG | *.md |

---

## Conclusion

`simple_k8s` would bring Kubernetes orchestration to the Simple Eiffel ecosystem, enabling:

1. **Programmatic cluster management** from Eiffel code
2. **Design by Contract** for resource specifications
3. **CI/CD integration** with simple_ci
4. **Production deployments** for Eiffel applications
5. **kubectl-like convenience** with KUBECTL_QUICK

**Recommended Priority**: HIGH (natural progression from simple_docker)
**Estimated Effort**: MEDIUM-HIGH (4 weeks for core, 2 weeks for advanced)
**Prerequisites**: simple_http, simple_json, simple_yaml (all exist)

---

## References

- [Kubernetes API Reference](https://kubernetes.io/docs/reference/kubernetes-api/)
- [Kubernetes C Client](https://github.com/kubernetes-client/c)
- [kubectl Quick Reference](https://kubernetes.io/docs/reference/kubectl/quick-reference/)
- [Authenticating to Kubernetes](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)
- [API Concepts](https://kubernetes.io/docs/reference/using-api/api-concepts/)
- [Client Libraries](https://kubernetes.io/docs/reference/using-api/client-libraries/)


---

## CI/CD Integration (Detailed)

### Overview

`simple_k8s` enables programmatic Kubernetes deployments from CI/CD pipelines.

### 1. GitHub Actions Integration

**Use Case**: Eiffel application deployed on push to main branch.

### 2. Blue/Green Deployments

**Concept**: Run two identical environments. Switch traffic atomically via service selector.

**Key Features**:
- Zero-downtime deployments
- Instant rollback (just switch selector)
- Requires 2x resources during deployment

### 3. Canary Deployments

**Concept**: Gradually shift traffic to new version (10% -> 25% -> 50% -> 100%).

### 4. Progressive Rollouts

**Concept**: Automated canary with metrics-based progression through stages.

### 5. CI/CD Strategy Comparison

| Strategy | Zero Downtime | Instant Rollback | Resource Usage | Complexity |
|----------|---------------|------------------|----------------|------------|
| Rolling Update | Yes | Minutes | 1x + surge | Low |
| Blue/Green | Yes | Instant | 2x | Medium |
| Canary | Yes | Fast | 1x + canary | Medium |
| Progressive | Yes | Automatic | 1x + canary | High |


---

## API Design Philosophy

### SDK Influence Analysis

| SDK | What to Adopt | What to Avoid |
|-----|---------------|---------------|
| **Go (client-go)** | Informer/watch patterns | Verbose boilerplate |
| **Python** | Simplicity, readability | Stringly-typed |
| **Rust (kube-rs)** | Fluent builders, strong typing | Async complexity |
| **kubectl CLI** | Command naming, one-liner patterns | N/A - UX gold standard |
| **.NET** | Fluent async patterns | Over-abstraction |

**Primary Models:** kube-rs (builders), kubectl (QUICK patterns), Python (simplicity)

### Pain Points Addressed

| Pain Point | simple_k8s Solution |
|------------|---------------------|
| Verbose NULL-heavy APIs | Fluent builders with sensible defaults |
| Stringly-typed configs | Strong typing: POD_SPEC, DEPLOYMENT_SPEC |
| Complex auth setup | make_with_kubeconfig just works |
| Cryptic errors | K8S_ERROR with reason, message, retry guidance |
| No retry logic | Built-in retry with exponential backoff |
| State confusion | DBC contracts enforce valid transitions |
| Callback hell | Synchronous by default, SCOOP for async |

---

## Three-Tier API Architecture

```
Tier 1: KUBECTL_QUICK / K8S_DEPLOY_QUICK (one-liners, patterns)
    |
    v
Tier 2: K8S_CLIENT + Spec Classes (full control, fluent builders)
    |
    v
Tier 3: Low-level HTTP (custom resources, raw API)
```

### Tier 1: KUBECTL_QUICK

kubectl-parity one-liners:
- run(name, image) - create pod
- create_deployment(name, image, replicas)
- scale(deployment, replicas)
- expose(deployment, port) / expose_lb()
- logs(pod) / logs_follow(pod)
- exec(pod, command)
- restart(deployment)
- rollback(deployment)
- pods / deployments / services - list queries
- apply(yaml_path)
- delete_pod/deployment/service(name)

### Tier 1.5: K8S_DEPLOY_QUICK

High-level deployment patterns:
- deploy_and_expose(name, image, port, replicas)
- rolling_update(deployment, new_image, timeout)
- rolling_update_or_rollback(deployment, new_image, timeout)
- blue_green_deploy(service, new_image, version)
- blue_green_switch(service, version)
- blue_green_rollback(service)
- canary_deploy(deployment, image, percentage)
- canary_promote(deployment)
- canary_abort(deployment)
- progressive_deploy(deployment, image)

### Tier 2: K8S_CLIENT

Full control with spec classes (POD_SPEC, DEPLOYMENT_SPEC, SERVICE_SPEC).

### Tier 3: Low-Level

Direct HTTP: api_get(), api_post(), api_patch(), api_delete(), custom_resource()


### Tier 1.6: K8S_CI_QUICK (CI Pipeline Operations)

Specifically designed for CI/CD pipelines with exit codes and failure handling.

**Features:**
- All operations exit(1) on failure (CI-friendly)
- Built-in timeouts
- Automatic rollback on failure
- Structured output for CI logs
- Environment variable configuration

**Operations:**
- deploy_or_fail(deployment, image)
- deploy_canary_or_fail(deployment, image, percentage)
- promote_or_fail(deployment)
- abort_or_fail(deployment)
- scale_or_fail(deployment, replicas)
- wait_ready_or_fail(deployment, timeout)
- smoke_test_or_fail(test_image, timeout)

**Usage in GitHub Actions:**
```yaml
- name: Deploy
  run: ./deployer deploy-or-fail myapp $${{ github.sha }}
```

**Usage in GitLab CI:**
```yaml
deploy:
  script:
    - ./deployer deploy-canary-or-fail myapp $$CI_COMMIT_SHA 10
    - ./deployer promote-or-fail myapp
```

**Class Summary:**
```
K8S_CI_QUICK
  deploy_or_fail (deployment, image)
  deploy_canary_or_fail (deployment, image, percentage)
  progressive_or_fail (deployment, image)
  promote_or_fail (deployment)
  abort_or_fail (deployment)
  rollback_or_fail (deployment)
  scale_or_fail (deployment, replicas)
  wait_ready_or_fail (deployment, timeout)
  smoke_test_or_fail (test_image, timeout)
  blue_green_or_fail (service, image, version)
```

All methods print structured status messages and call die(1) on failure.
