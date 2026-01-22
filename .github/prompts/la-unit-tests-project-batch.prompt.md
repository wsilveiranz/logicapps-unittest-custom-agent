---
name: la-unit-tests-project-batch
description: Execute project-wide unit test generation tasks (cases, implementation, test data) across all workflows in a Logic Apps Standard project.
---

## Responsibilities
- Use the discovery skill to enumerate workflows first.
- Apply the requested operation consistently per workflow.
- Produce a roll-up report of changes per workflow.

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
0. **Workspace Setup**
   - Ensure `plan/` folder exists at workspace root
   - Ensure `Tests/` folder exists at workspace root
   - If folders are not already in the VS Code workspace, add them using:
     - `code --add <workspace>/plan`
     - `code --add <workspace>/Tests`
   - This ensures test specs and test code are visible and accessible in the IDE

1. **Discovery Phase**
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
- **When creating diagrams** to visualize batch progress or workflow relationships, use **Mermaid format**

| Workflow | Spec | MockOutputs | Test Cases | Status |
|----------|------|-------------|------------|--------|
| workflow-1 | Created | 3 classes | 5 tests | ✅ Pass |
| workflow-2 | Updated | 2 classes | 3 tests | ✅ Pass |
