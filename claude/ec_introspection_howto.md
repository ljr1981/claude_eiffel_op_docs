# EC.EXE Introspection - How-To for Claude

**Purpose:** When and how to use ec.exe introspection options during Eiffel development.

---

## When to Use Introspection

### Use `-clients CLASS` when:
- **Before modifying a class** - Check who depends on it to assess impact
- **Refactoring** - Find all places that need updating when changing a class interface
- **Debugging compilation errors** - Understand why changes break other code
- **Understanding unfamiliar code** - See how a class fits into the larger system

```bash
"/c/Program Files/Eiffel Software/EiffelStudio 25.02 Standard/studio/spec/win64/bin/ec.exe" -config project.ecf -target target_tests -clients MY_CLASS
```

### Use `-suppliers CLASS` when:
- **Understanding dependencies** - See what a class needs to function
- **Breaking circular dependencies** - Identify what can be extracted
- **Planning refactoring** - Know what needs to remain available

### Use `-ancestors CLASS` when:
- **VHRC errors** - Check inheritance to see available features to rename/redefine
- **Understanding inheritance** - See the full parent chain
- **Fixing type conformance** - Verify a class conforms to expected type
- **Learning new simple_* library** - Quickly understand class hierarchy

```bash
"/c/Program Files/Eiffel Software/EiffelStudio 25.02 Standard/studio/spec/win64/bin/ec.exe" -config project.ecf -target target_tests -ancestors MY_CLASS
```

### Use `-descendants CLASS` when:
- **Before changing parent class** - See all classes that will be affected
- **Understanding polymorphism** - See all possible types that can substitute
- **API design** - Verify inheritance hierarchy as expected

### Use `-short CLASS` when:
- **Learning a class API** - Quick view of public interface without implementation
- **Writing client code** - See available features and their signatures
- **Documentation** - Generate interface documentation
- **Checking contracts** - See preconditions/postconditions

```bash
"/c/Program Files/Eiffel Software/EiffelStudio 25.02 Standard/studio/spec/win64/bin/ec.exe" -config project.ecf -target target_tests -short MY_CLASS
```

### Use `-flat CLASS` when:
- **Understanding full API** - See all features including inherited
- **Checking for feature conflicts** - See everything available in one view
- **Planning overrides** - See what features exist to potentially redefine

### Use `-flatshort CLASS` when:
- **Complete interface documentation** - All public features including inherited
- **API reference** - Full specification of what class provides

### Use `-callers CLASS.feature` when:
- **Before changing a feature** - Find all call sites that depend on current signature
- **Removing a feature** - Check nothing else uses it
- **Debugging** - Find where a feature is invoked from
- **Performance analysis** - Find hotspots by checking call frequency

```bash
"/c/Program Files/Eiffel Software/EiffelStudio 25.02 Standard/studio/spec/win64/bin/ec.exe" -config project.ecf -target target_tests -callers MY_CLASS.my_feature
```

### Use `-callees CLASS.feature` when:
- **Understanding feature behavior** - See what it delegates to
- **Debugging** - Trace execution flow
- **Testing** - Know what needs to be mocked/stubbed

### Use `-implementers CLASS.feature` when:
- **Checking polymorphic dispatch** - See all versions of a deferred feature
- **Finding redefinitions** - See where feature behavior differs
- **Understanding design** - See how interface is implemented

---

## Practical Workflow Examples

### Example 1: Before modifying SIMPLE_JSON

```bash
# Check who uses it
ec.exe -config simple_json.ecf -target simple_json_tests -clients SIMPLE_JSON

# See what it depends on
ec.exe -config simple_json.ecf -target simple_json_tests -suppliers SIMPLE_JSON

# Understand its interface
ec.exe -config simple_json.ecf -target simple_json_tests -short SIMPLE_JSON
```

### Example 2: Fixing VHRC error (feature not found)

```bash
# Check parent classes to see what features actually exist
ec.exe -config project.ecf -target tests -ancestors MY_CLASS

# See full flat view of available features
ec.exe -config project.ecf -target tests -flat PARENT_CLASS
```

### Example 3: Safe feature rename/remove

```bash
# Find all callers before changing
ec.exe -config project.ecf -target tests -callers MY_CLASS.old_feature_name

# Then check clients for import/type usages
ec.exe -config project.ecf -target tests -clients MY_CLASS
```

### Example 4: Learning new simple_* library

```bash
# Get the inheritance structure
ec.exe -config simple_csv.ecf -target simple_csv_tests -ancestors SIMPLE_CSV

# See the public interface
ec.exe -config simple_csv.ecf -target simple_csv_tests -short SIMPLE_CSV

# See complete interface including inherited
ec.exe -config simple_csv.ecf -target simple_csv_tests -flatshort SIMPLE_CSV
```

---

## Code Analysis

### Use `-code-analysis` when:
- Running quality checks before commits
- Finding potential issues in new code

```bash
ec.exe -config project.ecf -target tests -code-analysis -ca_class MY_CLASS
```

---

## Key Points

1. **Always use introspection BEFORE making changes** - Know the impact
2. **Use `-short` to learn APIs** - Much faster than reading source
3. **Use `-ancestors` when facing inheritance errors** - See what's actually available
4. **Use `-callers` before refactoring** - Never break callers unknowingly
5. **Use `-clients` to assess change impact** - Understand dependencies

---

## Command Template

```bash
"/c/Program Files/Eiffel Software/EiffelStudio 25.02 Standard/studio/spec/win64/bin/ec.exe" -config <project>.ecf -target <target>_tests <option> <CLASS[.feature]>
```

Options:
- `-clients CLASS` - Who uses this class
- `-suppliers CLASS` - What this class uses
- `-ancestors CLASS` - Inheritance chain (parents)
- `-descendants CLASS` - Who inherits from this
- `-short CLASS` - Public interface only
- `-flat CLASS` - All features including inherited
- `-flatshort CLASS` - Complete public interface
- `-callers CLASS.feature` - Who calls this feature
- `-callees CLASS.feature` - What this feature calls
- `-implementers CLASS.feature` - All implementations
