# E01: Identify Library Capabilities

## Objective
Document what the library provides that other libraries could use.

## Steps

### 1. Read Library README
```bash
cat /d/prod/{lib_name}/README.md
```

### 2. List Source Classes
```bash
ls /d/prod/{lib_name}/src/*.e
```

### 3. Document Capabilities

Create a capabilities list:

```markdown
## {lib_name} Capabilities

### Core Features
- Feature 1: {description}
- Feature 2: {description}
- Feature 3: {description}

### Key Classes
| Class | Purpose | Reuse Potential |
|-------|---------|-----------------|
| CLASS_1 | {purpose} | HIGH/MEDIUM/LOW |
| CLASS_2 | {purpose} | HIGH/MEDIUM/LOW |

### Use Cases for Other Libraries
1. {Use case 1} - Libraries that need {X} could use {feature}
2. {Use case 2} - Libraries that need {Y} could use {feature}
```

### 4. Identify Integration Patterns

Common patterns:
- **Serialization** - JSON, XML, YAML could use reflection for auto-serialization
- **Connection pooling** - Database, HTTP, MQ could use factory/pool patterns
- **Encoding** - Any text-processing library could use encoding conversion
- **Validation** - Any library accepting input could use validation
- **Logging** - Any library could use structured logging

## Output
Mental model of what the library provides and how it could help others.

## Verification Checkpoint
- [ ] README reviewed
- [ ] All source classes listed
- [ ] Capabilities documented
- [ ] Use cases identified

## Next Step
â†’ E02-SCAN-CANDIDATES.md
