# Eiffel Build and Test Rules

## CRITICAL: Directory Requirements

### ALWAYS change to the project directory before compiling or testing

**CORRECT:**
```bash
cd /d/prod/simple_json
"/c/Program Files/Eiffel Software/EiffelStudio 25.02 Standard/studio/spec/win64/bin/ec.exe" -batch -config simple_json.ecf -target simple_json_tests -c_compile
./EIFGENs/simple_json_tests/W_code/simple_json.exe
```

**WRONG:**
```bash
# DON'T do this - running from /d/prod causes EIFGENs confusion
cd /d/prod
"/c/Program Files/Eiffel Software/EiffelStudio 25.02 Standard/studio/spec/win64/bin/ec.exe" -batch -config /d/prod/simple_json/simple_json.ecf -target simple_json_tests -c_compile
/d/prod/EIFGENs/simple_json_tests/W_code/simple_json.exe
```

### ALWAYS use the test target of the ECF

When compiling for testing, use the `*_tests` target:
```bash
-target simple_json_tests
```

Not the library target:
```bash
# WRONG for testing
-target simple_json
```

### EIFGENs Location

When you cd to the project directory first:
- EIFGENs is created in: `PROJECT_DIR/EIFGENs/`
- Executable is at: `PROJECT_DIR/EIFGENs/TARGET_NAME/W_code/PROJECT.exe`

When you DON'T cd (WRONG):
- EIFGENs might be created in wrong location
- Can cause conflicts with other projects

## Build Command Template

```bash
# Step 1: Change to project directory
cd /d/prod/simple_XXXX

# Step 2: Compile with test target
"/c/Program Files/Eiffel Software/EiffelStudio 25.02 Standard/studio/spec/win64/bin/ec.exe" \
    -batch \
    -config simple_XXXX.ecf \
    -target simple_XXXX_tests \
    -c_compile

# Step 3: Run tests
./EIFGENs/simple_XXXX_tests/W_code/simple_XXXX.exe
```

## Environment Variables

Always export required environment variables before building:
```bash
export SIMPLE_JSON=/d/prod/simple_json
export SIMPLE_TESTING=/d/prod/simple_testing
export TESTING_EXT=/d/prod/testing_ext
# ... etc
```
