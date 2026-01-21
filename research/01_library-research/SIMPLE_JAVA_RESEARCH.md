# simple_java Research Report

**Date**: 2024-12-26
**Feasibility**: LOW
**Recommendation**: SKIP (use REST microservices instead)

## Executive Summary

Java interop via C FFI is technically possible through JNI (Java Native Interface) or the newer FFM (Foreign Function & Memory API), but the complexity and overhead make it impractical. The JVM runtime adds significant weight, and JNI is notoriously difficult. REST/gRPC microservices are the recommended approach for Java integration.

## Step 1: Problem Definition

**Goal**: Evaluate whether Eiffel can call Java libraries via C FFI.

**Potential Use Cases**:
- Enterprise Java libraries
- Android development (via JNI)
- Legacy Java codebases

## Step 2: Technical Research

### JNI (Legacy Approach)

JNI allows C code to:
1. Load the JVM into the process
2. Find and call Java methods
3. Pass data between C and Java

```c
#include <jni.h>

int main() {
    JavaVM *jvm;
    JNIEnv *env;
    JavaVMInitArgs vm_args;

    // Initialize JVM
    JavaVMOption options[1];
    options[0].optionString = "-Djava.class.path=./myapp.jar";
    vm_args.version = JNI_VERSION_1_8;
    vm_args.options = options;
    vm_args.nOptions = 1;

    JNI_CreateJavaVM(&jvm, (void**)&env, &vm_args);

    // Find class
    jclass cls = (*env)->FindClass(env, "com/example/MyClass");

    // Get method
    jmethodID mid = (*env)->GetStaticMethodID(env, cls, "add", "(II)I");

    // Call method
    jint result = (*env)->CallStaticIntMethod(env, cls, mid, 10, 20);

    (*jvm)->DestroyJavaVM(jvm);
    return 0;
}
```

### JNI Complexity

Every operation requires multiple steps:
1. Find class by name
2. Get method/field ID by signature
3. Call method with explicit type
4. Handle exceptions manually
5. Manage local/global references

Method signatures are cryptic:
- `(II)I` = takes two ints, returns int
- `(Ljava/lang/String;)V` = takes String, returns void
- `([B)Ljava/lang/String;` = takes byte[], returns String

### FFM (Project Panama) - Modern Approach

Java 22+ has FFM API for native interop:

```java
// Java calling C (opposite direction)
import java.lang.foreign.*;

void example() {
    Linker linker = Linker.nativeLinker();
    SymbolLookup lookup = SymbolLookup.loaderLookup();

    MethodHandle strlen = linker.downcallHandle(
        lookup.find("strlen").get(),
        FunctionDescriptor.of(ValueLayout.JAVA_LONG, ValueLayout.ADDRESS)
    );
}
```

FFM is primarily for Java calling C, not C calling Java.

### Eiffel-JNI Integration Pattern

```eiffel
class SIMPLE_JAVA

feature -- Initialization

    make
        local
            options: POINTER
        do
            create_jvm_options (options)
            jvm := c_create_jvm (options)
            env := c_get_env (jvm)
        ensure
            jvm_created: jvm /= default_pointer
        end

feature -- Class Operations

    find_class (name: STRING): detachable JAVA_CLASS
        local
            c_name: C_STRING
            cls: POINTER
        do
            create c_name.make (name.twin.replace_substring_all (".", "/"))
            cls := c_find_class (env, c_name.item)
            if cls /= default_pointer then
                create Result.make (env, cls)
            end
        end

feature {NONE} -- C Externals (verbose!)

    c_create_jvm (options: POINTER): POINTER
        external "C inline use <jni.h>"
        alias "[
            JavaVM *jvm;
            JNIEnv *env;
            JavaVMInitArgs vm_args;
            vm_args.version = JNI_VERSION_1_8;
            vm_args.options = (JavaVMOption*)$options;
            vm_args.nOptions = 1;
            JNI_CreateJavaVM(&jvm, (void**)&env, &vm_args);
            return jvm;
        ]"
        end

    -- ... dozens more externals needed

end
```

## Step 3: Ecosystem Compatibility

| Requirement | Challenge |
|-------------|-----------|
| SCOOP compatibility | JVM has its own threading model |
| Void safety | JNI returns can be null |
| DBC | Hard to express Java semantics in contracts |
| Windows support | Requires JDK installation |
| Complexity | JNI is notoriously error-prone |

## Step 4: Developer Pain Points

1. **JVM in-process** - 50-100MB memory overhead
2. **JNI complexity** - Method signatures, reference management
3. **Startup time** - JVM initialization takes 100-500ms
4. **Exception handling** - Must check for exceptions after every call
5. **Memory leaks** - Easy to leak local references
6. **Classpath issues** - JAR loading is finnicky
7. **Version compatibility** - JNI ABI varies between JDK versions

## Step 5: Alternative Approaches

### REST Microservices (Recommended)

```
[Eiffel App] <--HTTP--> [Java Service]
```

- Use simple_http to call Java services
- Spring Boot / Micronaut excel at REST
- Clean process isolation
- No JVM in Eiffel process

### gRPC (High Performance)

```
[Eiffel App] <--gRPC--> [Java Service]
```

- Use simple_grpc
- Protocol Buffers for type safety
- Efficient binary protocol
- Java gRPC is mature

### Process Execution (Simple)

```eiffel
-- Use simple_process to run Java JAR
result := process.execute ("java -jar myapp.jar --input data.json")
```

### Eiffel2Java Note

EiffelStudio includes Eiffel2Java, but it's:
- A transcompiler (Eiffel -> Java bytecode)
- Not FFI (doesn't call existing Java from Eiffel)
- Largely unmaintained

## Step 6: Design Decisions

**Decision: Do NOT build simple_java for JNI**

**Rationale**:
1. JNI complexity is prohibitive (30+ externals for basic functionality)
2. JVM overhead (memory, startup) in-process is undesirable
3. REST/gRPC are idiomatic for JVM interop
4. simple_http and simple_grpc already exist

### Recommended Patterns

| Use Case | Solution |
|----------|----------|
| Call Java libraries | REST microservice |
| High-performance RPC | gRPC |
| Run Java tools | simple_process |
| Enterprise integration | HTTP APIs |

## Step 7: Conclusion

**Feasibility**: LOW

**Recommendation**: SKIP simple_java JNI

**Rationale**:
- JNI is notoriously complex and error-prone
- JVM runtime overhead (50-100MB) in-process
- Method signatures are cryptic and type-unsafe
- REST/gRPC are the modern Java integration patterns
- Java ecosystem is designed for service-oriented architecture

**Alternative**: Use existing libraries
- `simple_http` for REST APIs to Java services
- `simple_grpc` for high-performance RPC
- `simple_process` to invoke Java JARs

**When JNI Might Make Sense**:
- Android NDK development (no alternative)
- Embedding JVM is a hard requirement
- Single-process constraint (rare)

---

*Research completed as part of cross-language interoperability investigation.*
