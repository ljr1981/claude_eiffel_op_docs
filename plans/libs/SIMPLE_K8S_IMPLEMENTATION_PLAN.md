# simple_k8s Implementation Plan

**Created:** 2025-12-17
**Status:** APPROVED
**Oracle Reference:** Query "kubernetes" or "k8s"

---

## Executive Summary

This plan delivers **simple_k8s** - a programmatic Kubernetes cluster management library for Eiffel. It builds on the existing simple_* ecosystem (simple_http, simple_json, simple_yaml) to provide Kubernetes orchestration capabilities.

**Deliverables:**
1. `simple_k8s` - Kubernetes API client library
2. Full ecosystem integration (README, docs, tests, GitHub repo, env var)

**Research Completed:** 14 vectors in `D:\prod\reference_docs\designs\SIMPLE_K8S_VISION.md`

---

## Part 1: Architecture Overview

### Design Principles

1. **Direct REST API** - No C library wrapping, use simple_http for HTTPS
2. **DBC Contracts** - Preconditions validate before API calls
3. **Fluent Builders** - POD_SPEC, DEPLOYMENT_SPEC use builder pattern
4. **Facade Pattern** - K8S_CLIENT is the main entry point
5. **Error Handling** - K8S_ERROR with detailed information

### Class Hierarchy

```
simple_k8s/
├── Core
│   ├── K8S_CLIENT           -- Main facade (connection, auth, operations)
│   ├── K8S_CONFIG           -- kubeconfig file parser
│   ├── K8S_AUTH             -- Authentication strategies
│   └── K8S_ERROR            -- Error handling
├── Resources
│   ├── K8S_POD              -- Pod resource
│   ├── K8S_DEPLOYMENT       -- Deployment resource
│   ├── K8S_SERVICE          -- Service resource
│   ├── K8S_NAMESPACE        -- Namespace resource
│   ├── K8S_CONFIGMAP        -- ConfigMap resource
│   └── K8S_SECRET           -- Secret resource
├── Specifications
│   ├── POD_SPEC             -- Pod configuration (fluent builder)
│   ├── DEPLOYMENT_SPEC      -- Deployment configuration
│   ├── SERVICE_SPEC         -- Service configuration
│   ├── SERVICE_PORT         -- Port specification
│   └── CONTAINER_SPEC       -- Container within pod
├── Utilities
│   ├── MANIFEST_BUILDER     -- YAML manifest generation
│   ├── KUBECTL_QUICK        -- One-liner convenience API
│   └── K8S_LABEL_SELECTOR   -- Label selector builder
└── Constants
    ├── K8S_API_VERSIONS     -- API version constants
    └── K8S_RESOURCE_TYPES   -- Resource type constants
```

---

## Part 2: Dependencies

### Required Libraries (All Exist)

| Library | Purpose | Version |
|---------|---------|---------|
| **simple_http** | HTTPS requests to K8s API | Current |
| **simple_json** | JSON parsing/generation | Current |
| **simple_yaml** | kubeconfig parsing | Current |
| **simple_file** | File operations | Current |
| **simple_base64** | Decode kubeconfig certs | Current |

### ECF Configuration

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<system xmlns="http://www.eiffel.com/developers/xml/configuration-1-23-0"
        name="simple_k8s"
        uuid="XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"
        library_target="simple_k8s">

    <target name="simple_k8s">
        <root all_classes="true"/>
        <file_rule>
            <exclude>/EIFGENs$</exclude>
            <exclude>/testing$</exclude>
        </file_rule>
        <option warning="warning" syntax="standard" manifest_array_type="mismatch_warning">
            <assertions precondition="true" postcondition="true" check="true" invariant="true"/>
        </option>
        <setting name="console_application" value="true"/>
        <setting name="concurrency" value="scoop"/>
        <setting name="void_safety" value="all"/>

        <!-- ISE Libraries -->
        <library name="base" location="$ISE_LIBRARY\library\base\base.ecf"/>
        <library name="time" location="$ISE_LIBRARY\library\time\time.ecf"/>

        <!-- Simple Libraries -->
        <library name="simple_http" location="$SIMPLE_EIFFEL\simple_http\simple_http.ecf"/>
        <library name="simple_json" location="$SIMPLE_EIFFEL\simple_json\simple_json.ecf"/>
        <library name="simple_yaml" location="$SIMPLE_EIFFEL\simple_yaml\simple_yaml.ecf"/>
        <library name="simple_file" location="$SIMPLE_EIFFEL\simple_file\simple_file.ecf"/>
        <library name="simple_base64" location="$SIMPLE_EIFFEL\simple_base64\simple_base64.ecf"/>
        <library name="simple_logger" location="$SIMPLE_EIFFEL\simple_logger\simple_logger.ecf"/>

        <cluster name="src" location=".\src\" recursive="true"/>
    </target>

    <target name="simple_k8s_tests" extends="simple_k8s">
        <root class="LIB_TESTS" feature="make"/>
        <library name="testing" location="$ISE_LIBRARY\library\testing\testing.ecf"/>
        <library name="simple_testing" location="$SIMPLE_EIFFEL\simple_testing\simple_testing.ecf"/>
        <cluster name="testing" location=".\testing\"/>
    </target>

</system>
```

---

## Part 3: Detailed Implementation

### Phase 1: Foundation (Week 1)

#### 1.1 K8S_CONFIG - kubeconfig Parser

```eiffel
class K8S_CONFIG

feature -- Access

    api_server: STRING
            -- Kubernetes API server URL.

    ca_certificate: detachable STRING
            -- CA certificate (PEM format, decoded from base64).

    client_certificate: detachable STRING
            -- Client certificate for mTLS.

    client_key: detachable STRING
            -- Client private key for mTLS.

    bearer_token: detachable STRING
            -- Bearer token for authentication.

    current_namespace: STRING
            -- Default namespace from context.

feature -- Factory

    make_from_file (a_path: STRING)
            -- Load kubeconfig from file.
        require
            file_exists: (create {RAW_FILE}.make (a_path)).exists
        end

    make_from_default
            -- Load from ~/.kube/config.
        end

    make_in_cluster
            -- Load in-cluster configuration.
        require
            in_cluster: is_in_cluster_environment
        end

feature -- Query

    is_valid: BOOLEAN
            -- Is configuration valid for connection?
        do
            Result := not api_server.is_empty and
                      (bearer_token /= Void or client_certificate /= Void)
        end

    is_in_cluster_environment: BOOLEAN
            -- Are we running inside a Kubernetes pod?
        do
            Result := (create {RAW_FILE}.make ("/var/run/secrets/kubernetes.io/serviceaccount/token")).exists
        end

end
```

#### 1.2 K8S_AUTH - Authentication Handler

```eiffel
class K8S_AUTH

feature -- Headers

    auth_headers (a_config: K8S_CONFIG): HASH_TABLE [STRING, STRING]
            -- Generate authentication headers.
        do
            create Result.make (2)
            if attached a_config.bearer_token as tok then
                Result.put ("Bearer " + tok, "Authorization")
            end
            Result.put ("application/json", "Accept")
            Result.put ("application/json", "Content-Type")
        end

feature -- TLS

    configure_tls (a_http: SIMPLE_HTTP; a_config: K8S_CONFIG)
            -- Configure TLS for HTTP client.
        do
            if attached a_config.ca_certificate as ca then
                a_http.set_ca_certificate (ca)
            end
            if attached a_config.client_certificate as cert and
               attached a_config.client_key as key then
                a_http.set_client_certificate (cert, key)
            end
        end

end
```

#### 1.3 K8S_ERROR - Error Handling

```eiffel
class K8S_ERROR

feature -- Access

    status_code: INTEGER
            -- HTTP status code (e.g., 404, 409, 500).

    reason: STRING
            -- Kubernetes reason (e.g., "NotFound", "AlreadyExists").

    message: STRING
            -- Human-readable error message.

    resource_kind: detachable STRING
            -- Resource type that caused error (e.g., "Pod").

    resource_name: detachable STRING
            -- Resource name that caused error.

feature -- Classification

    is_not_found: BOOLEAN
        do Result := status_code = 404 end

    is_conflict: BOOLEAN
        do Result := status_code = 409 end

    is_forbidden: BOOLEAN
        do Result := status_code = 403 end

    is_unauthorized: BOOLEAN
        do Result := status_code = 401 end

    is_server_error: BOOLEAN
        do Result := status_code >= 500 end

    is_retryable: BOOLEAN
            -- Should this error be retried?
        do
            Result := is_server_error or status_code = 429  -- Too Many Requests
        end

feature -- Factory

    make_from_json (a_json: JSON_VALUE)
            -- Parse Kubernetes error response.
        end

end
```

#### 1.4 K8S_CLIENT - Core Connection

```eiffel
class K8S_CLIENT

create
    make_with_kubeconfig,
    make_with_kubeconfig_path,
    make_in_cluster

feature {NONE} -- Initialization

    make_with_kubeconfig
            -- Connect using default kubeconfig.
        do
            create config.make_from_default
            create http.make
            create auth.make
            auth.configure_tls (http, config)
            create logger.make_with_level ({SIMPLE_LOGGER}.Level_info)
        ensure
            config_valid: config.is_valid
        end

feature -- Status

    is_connected: BOOLEAN
            -- Can we reach the API server?
        do
            Result := ping
        end

    server_version: STRING
            -- Kubernetes server version.
        local
            l_response: STRING
            l_json: JSON_VALUE
        do
            l_response := api_get ("/version")
            l_json := json.parse (l_response)
            Result := l_json.item ("gitVersion").string_value
        end

feature -- Namespaces

    namespaces: ARRAYED_LIST [K8S_NAMESPACE]
            -- List all namespaces.
        do
            Result := parse_namespace_list (api_get ("/api/v1/namespaces"))
        end

    current_namespace: STRING
            -- Current namespace from config.
        do
            Result := config.current_namespace
        end

feature -- Pod Operations

    list_pods (a_namespace: STRING): ARRAYED_LIST [K8S_POD]
            -- List all pods in namespace.
        require
            namespace_not_empty: not a_namespace.is_empty
        local
            l_url: STRING
        do
            l_url := "/api/v1/namespaces/" + a_namespace + "/pods"
            Result := parse_pod_list (api_get (l_url))
        end

    get_pod (a_name, a_namespace: STRING): detachable K8S_POD
            -- Get specific pod.
        require
            name_not_empty: not a_name.is_empty
        local
            l_url: STRING
        do
            l_url := "/api/v1/namespaces/" + a_namespace + "/pods/" + a_name
            Result := parse_pod (api_get (l_url))
        end

    create_pod (a_spec: POD_SPEC; a_namespace: STRING): K8S_POD
            -- Create pod from specification.
        require
            spec_valid: a_spec.is_valid
        local
            l_url: STRING
            l_body: STRING
        do
            l_url := "/api/v1/namespaces/" + a_namespace + "/pods"
            l_body := a_spec.to_json
            Result := parse_pod (api_post (l_url, l_body))
        ensure
            pod_created: Result /= Void
        end

    delete_pod (a_name, a_namespace: STRING)
            -- Delete pod.
        require
            name_not_empty: not a_name.is_empty
        local
            l_url: STRING
        do
            l_url := "/api/v1/namespaces/" + a_namespace + "/pods/" + a_name
            api_delete (l_url)
        end

    pod_logs (a_name, a_namespace: STRING): STRING
            -- Get pod logs.
        require
            name_not_empty: not a_name.is_empty
        local
            l_url: STRING
        do
            l_url := "/api/v1/namespaces/" + a_namespace + "/pods/" + a_name + "/log"
            Result := api_get (l_url)
        end

feature -- Deployment Operations

    list_deployments (a_namespace: STRING): ARRAYED_LIST [K8S_DEPLOYMENT]
        local
            l_url: STRING
        do
            l_url := "/apis/apps/v1/namespaces/" + a_namespace + "/deployments"
            Result := parse_deployment_list (api_get (l_url))
        end

    create_deployment (a_spec: DEPLOYMENT_SPEC; a_namespace: STRING): K8S_DEPLOYMENT
        require
            spec_valid: a_spec.is_valid
        local
            l_url: STRING
        do
            l_url := "/apis/apps/v1/namespaces/" + a_namespace + "/deployments"
            Result := parse_deployment (api_post (l_url, a_spec.to_json))
        end

    scale_deployment (a_name, a_namespace: STRING; a_replicas: INTEGER)
            -- Scale deployment to specified replicas.
        require
            replicas_valid: a_replicas >= 0
        local
            l_url: STRING
            l_patch: STRING
        do
            l_url := "/apis/apps/v1/namespaces/" + a_namespace + "/deployments/" + a_name + "/scale"
            l_patch := "{%"spec%":{%"replicas%":" + a_replicas.out + "}}"
            api_patch (l_url, l_patch)
        end

    rollout_restart (a_name, a_namespace: STRING)
            -- Trigger rolling restart.
        local
            l_url: STRING
            l_patch: STRING
            l_timestamp: STRING
        do
            l_timestamp := (create {DATE_TIME}.make_now).out
            l_url := "/apis/apps/v1/namespaces/" + a_namespace + "/deployments/" + a_name
            l_patch := "{%"spec%":{%"template%":{%"metadata%":{%"annotations%":{%"kubectl.kubernetes.io/restartedAt%":%"" + l_timestamp + "%"}}}}}"
            api_patch (l_url, l_patch)
        end

feature -- Service Operations

    list_services (a_namespace: STRING): ARRAYED_LIST [K8S_SERVICE]
        local
            l_url: STRING
        do
            l_url := "/api/v1/namespaces/" + a_namespace + "/services"
            Result := parse_service_list (api_get (l_url))
        end

    create_service (a_spec: SERVICE_SPEC; a_namespace: STRING): K8S_SERVICE
        require
            spec_valid: a_spec.is_valid
        local
            l_url: STRING
        do
            l_url := "/api/v1/namespaces/" + a_namespace + "/services"
            Result := parse_service (api_post (l_url, a_spec.to_json))
        end

feature {NONE} -- HTTP Operations

    api_get (a_path: STRING): STRING
            -- GET request to API server.
        local
            l_url: STRING
        do
            l_url := config.api_server + a_path
            Result := http.get (l_url, auth.auth_headers (config))
            check_response
        end

    api_post (a_path, a_body: STRING): STRING
            -- POST request to API server.
        local
            l_url: STRING
        do
            l_url := config.api_server + a_path
            Result := http.post (l_url, a_body, auth.auth_headers (config))
            check_response
        end

    api_delete (a_path: STRING)
            -- DELETE request to API server.
        local
            l_url: STRING
        do
            l_url := config.api_server + a_path
            http.delete (l_url, auth.auth_headers (config))
            check_response
        end

    api_patch (a_path, a_body: STRING): STRING
            -- PATCH request to API server.
        local
            l_url: STRING
            l_headers: HASH_TABLE [STRING, STRING]
        do
            l_url := config.api_server + a_path
            l_headers := auth.auth_headers (config)
            l_headers.put ("application/strategic-merge-patch+json", "Content-Type")
            Result := http.patch (l_url, a_body, l_headers)
            check_response
        end

feature {NONE} -- Implementation

    config: K8S_CONFIG
    http: SIMPLE_HTTP
    auth: K8S_AUTH
    logger: SIMPLE_LOGGER
    json: SIMPLE_JSON

    last_error: detachable K8S_ERROR

    has_error: BOOLEAN
        do Result := last_error /= Void end

    check_response
            -- Check HTTP response and set last_error if needed.
        do
            if http.last_status >= 400 then
                create last_error.make_from_json (json.parse (http.last_response))
            else
                last_error := Void
            end
        end

end
```

### Phase 2: Resource Specifications (Week 2)

#### 2.1 POD_SPEC

```eiffel
class POD_SPEC

create
    make

feature {NONE} -- Initialization

    make
        do
            create environment.make (10)
            create ports.make (5)
            create volumes.make (5)
            create labels.make (10)
            create annotations.make (10)
            restart_policy := "Always"
        end

feature -- Required

    name: STRING assign set_name
    image: STRING assign set_image

feature -- Optional

    namespace: STRING
    command: detachable ARRAY [STRING]
    args: detachable ARRAY [STRING]
    environment: HASH_TABLE [STRING, STRING]
    ports: ARRAYED_LIST [TUPLE [name: STRING; port: INTEGER]]
    volumes: ARRAYED_LIST [TUPLE [name, mount_path, source: STRING]]
    labels: HASH_TABLE [STRING, STRING]
    annotations: HASH_TABLE [STRING, STRING]
    restart_policy: STRING  -- Always, OnFailure, Never

    cpu_request: detachable STRING
    cpu_limit: detachable STRING
    memory_request: detachable STRING
    memory_limit: detachable STRING

feature -- Builder

    set_name (a_name: STRING): like Current
        require
            valid_name: is_valid_k8s_name (a_name)
        do
            name := a_name
            Result := Current
        ensure
            name_set: name.same_string (a_name)
        end

    set_image (a_image: STRING): like Current
        require
            not_empty: not a_image.is_empty
        do
            image := a_image
            Result := Current
        end

    add_env (a_key, a_value: STRING): like Current
        do
            environment.put (a_value, a_key)
            Result := Current
        end

    add_port (a_name: STRING; a_port: INTEGER): like Current
        require
            valid_port: a_port > 0 and a_port <= 65535
        do
            ports.extend ([a_name, a_port])
            Result := Current
        end

    add_volume (a_name, a_mount_path, a_source: STRING): like Current
        do
            volumes.extend ([a_name, a_mount_path, a_source])
            Result := Current
        end

    add_label (a_key, a_value: STRING): like Current
        do
            labels.put (a_value, a_key)
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
        do
            Result := name /= Void and then not name.is_empty and
                      image /= Void and then not image.is_empty
        end

    is_valid_k8s_name (a_name: STRING): BOOLEAN
            -- Valid DNS subdomain name?
        do
            Result := not a_name.is_empty and a_name.count <= 253
            -- Additional: lowercase, alphanumeric, hyphens
        end

feature -- Output

    to_json: STRING
            -- Generate JSON for API request.
        require
            valid: is_valid
        local
            l_json: JSON_OBJECT
        do
            create l_json.make
            l_json.put_string ("v1", "apiVersion")
            l_json.put_string ("Pod", "kind")
            -- ... build full structure
            Result := l_json.representation
        end

invariant
    restart_valid: restart_policy.same_string ("Always") or
                   restart_policy.same_string ("OnFailure") or
                   restart_policy.same_string ("Never")

end
```

### Phase 3: Convenience API (Week 3)

#### 3.1 KUBECTL_QUICK

```eiffel
class KUBECTL_QUICK

create
    make

feature {NONE} -- Initialization

    make (a_client: K8S_CLIENT)
        do
            client := a_client
            namespace := a_client.current_namespace
        end

feature -- Quick Commands

    run (a_name, a_image: STRING): K8S_POD
            -- Like: kubectl run NAME --image=IMAGE
        do
            Result := client.create_pod (
                create {POD_SPEC}.make.set_name (a_name).set_image (a_image),
                namespace
            )
        end

    scale (a_deployment: STRING; a_replicas: INTEGER)
            -- Like: kubectl scale deployment NAME --replicas=N
        do
            client.scale_deployment (a_deployment, namespace, a_replicas)
        end

    logs (a_pod: STRING): STRING
            -- Like: kubectl logs POD
        do
            Result := client.pod_logs (a_pod, namespace)
        end

    delete_pod (a_name: STRING)
            -- Like: kubectl delete pod NAME
        do
            client.delete_pod (a_name, namespace)
        end

    get_pods: ARRAYED_LIST [K8S_POD]
            -- Like: kubectl get pods
        do
            Result := client.list_pods (namespace)
        end

    describe_pod (a_name: STRING): STRING
            -- Like: kubectl describe pod NAME
        local
            l_pod: detachable K8S_POD
        do
            l_pod := client.get_pod (a_name, namespace)
            if l_pod /= Void then
                Result := l_pod.describe
            else
                Result := "Pod not found: " + a_name
            end
        end

    set_namespace (a_namespace: STRING)
            -- Like: kubectl config set-context --current --namespace=NS
        do
            namespace := a_namespace
        end

feature {NONE} -- Implementation

    client: K8S_CLIENT
    namespace: STRING

end
```

---

## Part 4: File Structure

```
D:\prod\simple_k8s\
├── src\
│   ├── core\
│   │   ├── k8s_client.e
│   │   ├── k8s_config.e
│   │   ├── k8s_auth.e
│   │   └── k8s_error.e
│   ├── resources\
│   │   ├── k8s_pod.e
│   │   ├── k8s_deployment.e
│   │   ├── k8s_service.e
│   │   ├── k8s_namespace.e
│   │   ├── k8s_configmap.e
│   │   └── k8s_secret.e
│   ├── specs\
│   │   ├── pod_spec.e
│   │   ├── deployment_spec.e
│   │   ├── service_spec.e
│   │   ├── service_port.e
│   │   └── container_spec.e
│   ├── util\
│   │   ├── kubectl_quick.e
│   │   ├── manifest_builder.e
│   │   └── k8s_label_selector.e
│   └── constants\
│       ├── k8s_api_versions.e
│       └── k8s_resource_types.e
├── testing\
│   ├── lib_tests.e
│   ├── test_config.e
│   ├── test_pod_operations.e
│   ├── test_deployment_operations.e
│   └── test_service_operations.e
├── docs\
│   ├── index.html
│   ├── user-guide.html
│   ├── api-reference.html
│   ├── architecture.html
│   └── cookbook.html
├── simple_k8s.ecf
├── README.md
├── CHANGELOG.md
└── package.json
```

---

## Part 5: Test Plan

### Test Prerequisites

- **Kubernetes cluster** available (minikube, kind, Docker Desktop, or remote)
- **kubeconfig** configured at `~/.kube/config`
- **Namespace** `simple-k8s-test` created for tests

### Test Categories

| Category | Tests | Coverage |
|----------|-------|----------|
| Config | 5 | kubeconfig parsing, in-cluster detection |
| Connection | 3 | Ping, version, namespace list |
| Pods | 10 | CRUD, logs, status |
| Deployments | 8 | CRUD, scale, rollout |
| Services | 5 | CRUD, types |
| Specs | 8 | Validation, JSON generation |
| Quick | 5 | Convenience API |
| **Total** | **44** | |

### Test Classes

```eiffel
class TEST_CONFIG

inherit
    TEST_SET_BASE

feature -- Tests

    test_load_kubeconfig
        local
            l_config: K8S_CONFIG
        do
            create l_config.make_from_default
            assert ("config valid", l_config.is_valid)
            assert ("has server", not l_config.api_server.is_empty)
        end

    test_parse_kubeconfig_yaml
        -- Test parsing of clusters, contexts, users
        end

    test_in_cluster_detection
        local
            l_config: K8S_CONFIG
        do
            -- Should return False when not in pod
            assert ("not in cluster", not (create {K8S_CONFIG}).is_in_cluster_environment)
        end

end
```

---

## Part 6: Ecosystem Integration

### GitHub Repository

```bash
"/c/Program Files/GitHub CLI/gh.exe" repo create simple-eiffel/simple_k8s \
    --public \
    --description "Kubernetes cluster orchestration for Eiffel: deploy, scale, manage workloads programmatically"
```

### Environment Variable

```bash
# Unix
export SIMPLE_K8S=/d/prod/simple_k8s

# Windows (persistent)
setx SIMPLE_K8S D:\prod\simple_k8s
```

### Oracle Registration

```bash
oracle-cli.exe log info simple_k8s "Library created: Kubernetes cluster orchestration"
```

### Documentation Site

Update `https://simple-eiffel.github.io/`:
- Add simple_k8s to library list
- Category: Infrastructure
- Tier: Advanced

---

## Part 7: Timeline

### Week 1: Foundation
- Day 1-2: K8S_CONFIG (kubeconfig parser)
- Day 3: K8S_AUTH (auth strategies)
- Day 4-5: K8S_CLIENT core (connection, ping, version)

### Week 2: Core Resources
- Day 1-2: POD_SPEC, K8S_POD
- Day 3: Pod CRUD operations
- Day 4-5: Tests for pods

### Week 3: Deployments & Services
- Day 1-2: DEPLOYMENT_SPEC, K8S_DEPLOYMENT
- Day 3: SERVICE_SPEC, K8S_SERVICE
- Day 4-5: Scale, rollout, service operations

### Week 4: Polish
- Day 1: KUBECTL_QUICK convenience API
- Day 2: MANIFEST_BUILDER
- Day 3: Documentation (IUARC 5-doc)
- Day 4: README, CHANGELOG
- Day 5: GitHub push, ecosystem integration

---

## Part 8: Success Criteria

| Metric | Target | Status |
|--------|--------|--------|
| Compiles | Windows + Linux | - |
| kubeconfig support | Full parsing | - |
| Pod operations | CRUD + logs | - |
| Deployment operations | CRUD + scale + rollout | - |
| Service operations | CRUD | - |
| Tests passing | 44+ tests | - |
| Documentation | IUARC 5-doc | - |
| GitHub repo | Created + pushed | - |

---

## Part 9: Risk Mitigation

| Risk | Impact | Mitigation |
|------|--------|------------|
| **simple_http TLS issues** | HIGH | Test early, may need cert handling fixes |
| **API version changes** | MEDIUM | Abstract API versions, support negotiation |
| **Complex auth scenarios** | MEDIUM | Start with kubeconfig + bearer token |
| **Watch streaming** | LOW (Phase 7) | Defer to later phase |
| **RBAC restrictions** | LOW | Document required permissions |

---

## Part 10: Future Phases (Post-MVP)

| Phase | Scope | Priority |
|-------|-------|----------|
| **7** | Watch/streaming for real-time updates | Medium |
| **8** | Ingress, NetworkPolicy | Medium |
| **9** | RBAC resources (Roles, RoleBindings) | Low |
| **10** | Custom Resource Definitions (CRDs) | Low |
| **11** | Helm chart support | Low |
| **12** | Multi-cluster management | Low |

---

## Execution Command

After plan approval:
```bash
# Create directory structure
mkdir -p /d/prod/simple_k8s/{src/{core,resources,specs,util,constants},testing,docs}

# Start with K8S_CONFIG
cd /d/prod/simple_k8s
```

---

## Critical File References

| File | Purpose |
|------|---------|
| `D:\prod\reference_docs\designs\SIMPLE_K8S_VISION.md` | Research findings |
| `D:\prod\simple_docker\src\docker_client.e` | Pattern reference |
| `D:\prod\simple_http\src\simple_http.e` | HTTP client |
| `D:\prod\simple_json\src\core\simple_json.e` | JSON parsing |
| `D:\prod\simple_yaml\src\simple_yaml.e` | YAML parsing |
| `D:\prod\simple_testing\src\test_set_base.e` | Test base class |


### K8S_CI_QUICK Addition

Add to Phase 4 (Week 4):

| Class | Purpose | Priority |
|-------|---------|----------|
| K8S_CI_QUICK | CI pipeline operations with exit codes | P1 |

File: src/util/k8s_ci_quick.e
