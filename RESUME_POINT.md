# Resume Point

**Date:** 2025-12-08
**Last Session:** AutoTest fixes, ECF UUID deduplication, installer improvements

---

## Current State

### simple_setup (COMPLETE)
- **Location:** `D:\prod\simple_setup`
- **Status:** Fully functional, installer v1.0.3 generated
- **GitHub Repo:** https://github.com/simple-eiffel/simple_setup

#### Recent Improvements (This Session)
1. **Installer EIFGENs exclusion fixed** - Changed `*\EIFGENs\*` to `EIFGENs\*` (Inno Setup matches from relative path)
2. **Added logo branding** - `WizardImageFile=logo-tall.png`, `WizardSmallImageFile=logo-small.png`
3. **Directory selection enabled** - Added `DisableDirPage=no` to [Setup] section
4. **Version:** 1.0.3 installer at `D:\prod\simple_setup\output\simple_ecosystem_1.0.3_setup.exe`

### AutoTest Discovery (FIXED)
Tests were not appearing in EiffelStudio AutoTest tool. Root causes and fixes:

1. **Wrong testing note** - 144 test files had `testing: "type/manual"` which hides from AutoTest
   - Fixed: Changed to `testing: "covers/{CLASS}.feature"` format

2. **Missing testing.ecf library** - Test targets had `simple_testing` but not ISE's `testing.ecf`
   - Fixed: Added `<library name="testing" location="$ISE_LIBRARY\library\testing\testing.ecf"/>` to 40 ECFs

### ECF UUID Deduplication (FIXED)
- Found duplicate UUID `A1B2C3D4-E5F6-7890-ABCD-EF1234567890` shared by simple_ci and simple_validation
- Fixed: Changed simple_ci UUID to `C1D2E3F4-5A6B-7C8D-9E0F-A1B2C3D4E5F6`
- All 40+ ECFs now have unique UUIDs

### Compilation Fixes (FIXED)
| File | Issue | Fix |
|------|-------|-----|
| `simple_jwt\tests\simple_jwt_test_set.e:566` | Unknown `assert_integers_64_equal` | Changed to `assert ("user_id equals 12345", user_id = 12345)` |
| `simple_logger\src\simple_logger.e` | Unused `l_writer` | Removed local declaration |
| `simple_datetime\src\simple_date.e` | Unused `l_jan1_dow` | Removed local declaration |
| `simple_datetime\src\simple_time.e` | Unused `l_time_part` | Removed local declaration |

---

## Helper Scripts Created

### reset_env.ps1
Resets all 44 SIMPLE_* environment variables to `D:\prod`:
```powershell
D:\prod\simple_setup\reset_env.ps1
```

### fix_ecfs.ps1
Adds `testing.ecf` library to ECFs that have `simple_testing` but missing ISE testing:
```powershell
D:\prod\simple_setup\fix_ecfs.ps1
```

---

## GitHub Organization
- **Organization:** https://github.com/simple-eiffel
- **Plan:** Team ($4/month)
- **Repos:** 57 simple_* libraries transferred

---

## Environment Variables

All set to `D:\prod\<library_name>`:
```
SIMPLE_BASE64, SIMPLE_HASH, SIMPLE_UUID, SIMPLE_RANDOMIZER, SIMPLE_JSON,
SIMPLE_XML, SIMPLE_CSV, SIMPLE_MARKDOWN, SIMPLE_DATETIME, SIMPLE_VALIDATION,
SIMPLE_REGEX, SIMPLE_PROCESS, SIMPLE_LOGGER, SIMPLE_HTMX, SIMPLE_ENV,
SIMPLE_SYSTEM, SIMPLE_CONSOLE, SIMPLE_CLIPBOARD, SIMPLE_REGISTRY, SIMPLE_MMAP,
SIMPLE_IPC, SIMPLE_WATCHER, SIMPLE_WIN32_API, SIMPLE_CACHE, SIMPLE_TEMPLATE,
SIMPLE_JWT, SIMPLE_CORS, SIMPLE_RATE_LIMITER, SIMPLE_SMTP, SIMPLE_SQL,
SIMPLE_WEBSOCKET, SIMPLE_HTTP, SIMPLE_ENCRYPTION, SIMPLE_CONFIG, SIMPLE_PDF,
SIMPLE_TESTING, SIMPLE_FOUNDATION_API, SIMPLE_SERVICE_API, SIMPLE_WEB,
SIMPLE_APP_API, SIMPLE_ALPINE, SIMPLE_AI_CLIENT, SIMPLE_GUI_DESIGNER, SIMPLE_SETUP
```

---

## Commands to Resume

### Build simple_setup
```bash
cd /d/prod/simple_setup
"/c/Program Files/Eiffel Software/EiffelStudio 25.02 Standard/studio/spec/win64/bin/ec.exe" \
  -batch -config simple_setup.ecf -target simple_setup -c_compile
```

### Build Installer
```bash
./EIFGENs/simple_setup/W_code/simple_setup.exe build-installer --version=1.0.3
```

### Test simple_setup CLI
```bash
./EIFGENs/simple_setup/W_code/simple_setup.exe help
./EIFGENs/simple_setup/W_code/simple_setup.exe list
./EIFGENs/simple_setup/W_code/simple_setup.exe status
```

---

## Minor TODOs (Optional)

1. **Inno Script Warning:** `IsTaskSelected` should be `WizardIsTaskSelected`
   - In `generate_code` feature of `sst_inno_generator.e`

2. **Missing Libraries:** Create simple_http, simple_encryption, simple_config
   - Or remove from manifest if not needed

---

## Key Reference Files

| Need | Location |
|------|----------|
| simple_setup source | `D:\prod\simple_setup\src\` |
| Library manifest | `D:\prod\simple_setup\src\sst_manifest.e` |
| Inno generator | `D:\prod\simple_setup\src\sst_inno_generator.e` |
| Generated installer | `D:\prod\simple_setup\output\` |
| Reset env script | `D:\prod\simple_setup\reset_env.ps1` |
| Fix ECFs script | `D:\prod\simple_setup\fix_ecfs.ps1` |

---

## Library Layers (43 total)

| Layer | Count | Examples |
|-------|-------|----------|
| Foundation | 14 | json, csv, hash, uuid, regex, datetime, xml |
| Platform | 9 | env, console, registry, win32_api, mmap, ipc |
| Service | 12 | sql, smtp, jwt, cache, template, pdf, http |
| API | 8 | testing, foundation_api, service_api, web, app_api |

---

*Last updated: 2025-12-08 - AutoTest fixed, ECF UUIDs deduplicated, installer v1.0.3*
