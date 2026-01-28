---
name: la-unit-tests-project-batch
description: Execute project-wide unit test generation tasks (cases, implementation, test data) across all workflows in a Logic Apps Standard project.
---

## Responsibilities
- Use the discovery skill to enumerate workflows first.
- Apply the requested operation consistently per workflow.
- Produce a roll-up report of changes per workflow.

## Pre-Execution Checklist (per workflow)
**AGENT MUST verify these conditions before implementing tests for each workflow:**

1. ✅ **Workflow Exists**: Confirm `workflow.json` exists in `<LogicAppsProject>/<workflow-name>/`
2. ✅ **Test Project Structure**: Verify `Tests/LogicApps/<workflow-name>/` exists
3. ✅ **MockOutput Classes**: Verify MockOutput classes exist in `Tests/LogicApps/<workflow-name>/MockOutputs/`
4. ✅ **Configuration File**: Check for `testSettings.config` in workflow test folder
5. ✅ **Project File**: Verify `LogicApps.Tests.csproj` exists with correct PackageReferences

### Decision Tree:
```
1. DISCOVER all workflows first
2. CHECK scaffolding for ALL workflows
3. IF any workflow missing scaffolding:
   → STOP immediately
   → Generate Scaffolding Report (see below)
   → Wait for user to complete scaffolding
   → DO NOT proceed with batch until all scaffolding is in place
4. ONLY when all scaffolding verified:
   → PROCEED with batch operations
```

### Scaffolding Report Format:
When scaffolding is missing, generate this report BEFORE any batch processing:

```markdown
## ⚠️ Scaffolding Required - Batch Cannot Proceed

The following workflows are missing test scaffolding:

| Workflow | Missing Components |
|----------|--------------------|
| workflow-1 | Test folder, MockOutputs, testSettings.config |
| workflow-2 | MockOutputs |

### Action Required:
For each workflow listed above:
1. Open the workflow in the local Logic Apps Designer
2. From the top menu, select **`Create Unit Test`**
3. This will automatically generate the test project structure

### After Scaffolding:
Once all workflows have been scaffolded, notify the agent to restart the batch operation.
```

**CRITICAL**: Do NOT proceed with any test implementation until ALL workflows have scaffolding in place.

## Project Structure (Extension Pattern)
```
<workspace>/
├── <LogicAppsProject>/           # Logic Apps Standard project
│   ├── host.json
│   ├── connections.json
│   ├── local.settings.json
│   └── <workflow-name>/
│       └── workflow.json
├── plan/                          # Test specifications (at workspace root)
│   └── <workflow-name>-testplan.md
└── Tests/                         # Unit test project
    ├── Tests.sln
    └── LogicApps/
        ├── LogicApps.csproj
        ├── TestExecutor.cs
        └── <workflow-name>/
            ├── testSettings.config
            ├── MockOutputs/
            │   └── *.cs
            └── TC<##>_<Scenario>/
                └── TC<##>_<Scenario>.cs
```

## Batch Operation Workflow

### 0. Workspace Setup
Verify these folders exist and are in VS Code workspace (use `code --add <path>` if needed):
- `<workspace>/plan/` - Test specifications
- `<workspace>/Tests/` - Test projects

### 1. Discovery Phase**
   - Scan for `workflow.json` files
   - Build workflow inventory with trigger/action details
   - Identify mockable dependencies

2. **Spec Generation Phase** (per workflow)
   - Create/update `<workspace>/plan/<workflow-name>-testplan.md` (at workspace root, not inside the Logic Apps project folder)
   - Define test case catalog
   - Document mock requirements

3. **Implementation Phase** (per workflow)
   - Create `Tests/LogicApps/<workflow-name>/` structure
   - Generate `testSettings.config`
   - Generate `MockOutputs/` classes
   - Generate test case files in subfolders

4. **Validation Phase**
   - Run `dotnet build` on Tests project
   - Run `dotnet test` to verify all tests pass
   - Report any failures

## testSettings.config Template
```xml
<?xml version="1.0" encoding="utf-8"?>
<testSettings>
  <WorkspacePath>../../../../../</WorkspacePath>
  <LogicAppName><LogicAppsProjectName></LogicAppName>
  <WorkflowName><workflow-name></WorkflowName>
</testSettings>
```

## Output
- Summary table: workflow -> artifacts created/updated
- Any skipped workflows + reason
- Build/test results

| Workflow | Spec | MockOutputs | Test Cases | Status |
|----------|------|-------------|------------|--------|
| workflow-1 | Created | 3 classes | 5 tests | ✅ Pass |
| workflow-2 | Updated | 2 classes | 3 tests | ✅ Pass |
