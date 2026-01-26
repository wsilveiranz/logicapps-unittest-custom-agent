---
name: la-unit-tests-implement
description: Implement MSTest unit tests for Logic Apps Standard workflows using the Automated Test SDK (Microsoft.Azure.Workflows.WebJobs.Tests.Extension) on net8.0.
---

## Pre-Execution Checklist
**AGENT MUST verify these conditions before implementing tests:**

1. ✅ **Workspace Structure**: Confirm `Tests/LogicApps/<workflow-name>/` exists
2. ✅ **MockOutput Classes**: Verify all required MockOutput classes exist in `Tests/LogicApps/<workflow-name>/MockOutputs/`
3. ✅ **Configuration Files**: Check for `testSettings.config` in workflow test folder
4. ✅ **Project File**: Verify `LogicApps.Tests.csproj` exists with correct PackageReferences

**If any condition fails:**
- **STOP** and inform the user that manual scaffolding is required
- Direct user to run `Create Unit Test` from Logic Apps Designer for the workflow
- **DO NOT** proceed with test implementation until scaffolding is complete

## Workspace Setup
**AGENT SHOULD verify** the following folders exist and are added to the VS Code workspace:

1. **Plan folder**: Ensure `<workspace>/plan/` exists at workspace root
   - If not already in the VS Code workspace, add it using: `code --add <workspace>/plan`
2. **Tests folder**: Create `<workspace>/Tests/` if it doesn't exist
   - If not already in the VS Code workspace, add it using: `code --add <workspace>/Tests`

This ensures all test artifacts are visible and accessible in the IDE.

## Critical: Scaffolding the Test Project for each Workflow
**AGENT MUST** verify the test project scaffolding is in place before implementing tests.

### How to Check Programmatically:
Use file system tools to verify these paths exist:
- `<workspace>/Tests/LogicApps/LogicApps.Tests.csproj` (project file)
- `<workspace>/Tests/LogicApps/<workflow-name>/` (workflow test folder)
- `<workspace>/Tests/LogicApps/<workflow-name>/MockOutputs/` (MockOutput classes)
- `<workspace>/Tests/LogicApps/<workflow-name>/testSettings.config` (configuration file)

### Decision Tree:
```
═══════════════════════════════════════════════════════════════
CHECK 1: Does the test project exist?
═══════════════════════════════════════════════════════════════
LogicApps.Tests.csproj exists?
├── YES → Continue to Check 2
└── NO  → STOP - Full scaffolding required
          Provide Manual Scaffolding Instructions (below)
          Wait for user confirmation before proceeding

═══════════════════════════════════════════════════════════════
CHECK 2: Does the workflow-specific folder exist?
═══════════════════════════════════════════════════════════════
<workflow-name>/ folder exists under Tests/LogicApps/?
├── YES → Continue to Check 3
└── NO  → STOP - Workflow scaffolding required
          ⚠️ Even though the test project exists, the workflow
          folder must be scaffolded using the Logic Apps extension.
          
          AGENT MUST:
          1. Inform user: "The test project exists, but scaffolding
             for workflow '<workflow-name>' is missing."
          2. Provide Manual Scaffolding Instructions (below)
          3. Tell user: "After scaffolding completes, let me know
             and I will continue with test implementation."
          4. WAIT for user confirmation before proceeding
          5. After confirmation, re-verify the folder exists

═══════════════════════════════════════════════════════════════
CHECK 3: Are all required files in place?
═══════════════════════════════════════════════════════════════
MockOutputs/ folder AND testSettings.config exist?
├── YES → ✅ PROCEED with test implementation
└── NO  → STOP - Incomplete scaffolding
          Ask user to re-run scaffolding for the workflow
```

### Manual Scaffolding Instructions (for user):
**AGENT MUST provide these instructions when scaffolding is missing:**

1. Open the workflow `<workflow-name>` in the local Logic Apps Designer
2. From the top menu, select **`Create Unit Test`**
3. This will automatically generate:
   - Test project structure under `Tests/LogicApps/`
   - MockOutput classes in `<workflow-name>/MockOutputs/`
   - Configuration file `testSettings.config`
4. After scaffolding completes, notify the agent to continue

**Note:** The scaffolding process may create placeholder test folders (e.g., `unittest1`). The agent should ask the user if these can be removed after implementing the actual test cases.

## ⚠️ CRITICAL CONSTRAINTS SUMMARY
**AGENT MUST enforce these constraints in all generated code:**

1. **ActionMock Constructors**: Use separate constructors for success (with `outputs`) vs failure (with `error`). **NEVER** pass outputs to a failed action mock.

2. **ServiceProvider Output Format**: Must include exactly these properties with `[JsonProperty]` attributes:
   - `body` (JToken)
   - `statusCode` (int) 
   - `headers` (JObject)
   - **NO** unexpected properties like `content` or `properties`

3. **Trigger Body Wrapper**: Trigger outputs **MUST** wrap data in a `body` property for `@triggerBody()` expressions to work.

4. **UnitTestExecutor Constructor**: Always use 4-parameter constructor. Pass `null` for `parametersFilePath` if not needed.

5. **Result Properties**: Assert using `result.Status` and `result.Actions["name"]`, **NOT** `WorkflowStatus` or `ActionResults`.

6. **MockOutput Classes**: Must exist in `Tests/LogicApps/<workflow-name>/MockOutputs/` before implementation.

*Detailed explanations of each constraint are provided in sections below.*

## Defaults (must enforce)
- Target framework: `net8.0`
- Inline PackageReferences:
  ```xml
  <PackageReference Include="Microsoft.Azure.Workflows.WebJobs.Tests.Extension" Version="1.*" />
  <PackageReference Include="MSTest.TestAdapter" Version="3.2.0" />
  <PackageReference Include="MSTest.TestFramework" Version="3.2.0" />
  <PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.9.0" />
  ```

## MSTest Annotations (required)
**AGENT MUST include** the following annotations on every test class and method:

### Class-Level Annotations:
```csharp
[TestClass]
[TestCategory("Workflow")]           // Always include for workflow tests
[TestCategory("<workflow-name>")]    // e.g., "la-sb-process-order"
public class TC01_TestClassName
{
    // ...
}
```

### Method-Level Annotations:
```csharp
[TestMethod]
[Priority(1)]                                    // 1=Critical, 2=High, 3=Normal
[Description("Brief description of test intent")]
[TestProperty("TestCaseId", "TC01")]
[TestProperty("Category", "HappyPath")]          // or "ErrorHandling", "DataVerification"
public async Task TC01_MethodName()
{
    // ...
}
```

### Priority Guidelines:
| Priority | Usage |
|----------|-------|
| 1 | Happy path, critical business flows |
| 2 | Error handling, edge cases |
| 3 | Data verification, non-critical scenarios |

### Category Values for TestProperty:
- `HappyPath` - Normal successful execution paths
- `ErrorHandling` - Failure and exception scenarios
- `DataVerification` - Input/output data validation
- `BoundaryConditions` - Edge cases and limits

## SDK Core Classes (from Microsoft.Azure.Workflows.UnitTesting)
| Class | Purpose | Notes | Documentation |
|-------|---------|-------|------|
| `UnitTestExecutor` | Main executor; 4-param constructor | `new UnitTestExecutor(workflowFilePath, connectionsFilePath, parametersFilePath, localSettingsFilePath)` - use `null` for parametersFilePath if not needed | [MS Learn](https://learn.microsoft.com/en-us/azure/logic-apps/testing-framework/unit-test-executor-class-definition) |
| `TriggerMock` | Mock for the workflow trigger | Constructor: `TriggerMock(TestWorkflowStatus status, string name, MockOutput outputs)` | [MS Learn](https://learn.microsoft.com/en-us/azure/logic-apps/testing-framework/trigger-mock-class-definition) |
| `ActionMock` | Mock for workflow actions | **Two constructors** - see Mock Patterns section | [MS Learn](https://learn.microsoft.com/en-us/azure/logic-apps/testing-framework/action-mock-class-definition) |
| `MockOutput` | Base class for mock outputs | Serializes to JSON; use `[JsonProperty]` for property naming | |
| `TestWorkflowStatus` | Enum | `Succeeded`, `Failed`, `Skipped`, `TimedOut` | [MS Learn](https://learn.microsoft.com/en-us/azure/logic-apps/testing-framework/test-workflow-status-enum-definition) |
| `TestErrorInfo` | Error info for failed actions | Constructor: `TestErrorInfo(ErrorResponseCode code, string message, TestErrorInfo[] details, TestErrorResponseAdditionalInfo[] additionalInfo)` | [MS Learn](https://learn.microsoft.com/en-us/azure/logic-apps/testing-framework/test-error-info-class-definition) |
| `ErrorResponseCode` | Error code enum | In `Microsoft.Azure.Workflows.Common.ErrorResponses` namespace | |

## Critical: Class documentation
Refer to the official Microsoft Learn documentation for each SDK class for detailed usage and examples (links provided in the table above).

## Critical: MockOutput Classes
**AGENT MUST verify** all required MockOutput classes are available within the test project under `Tests/LogicApps/<workflow-name>/MockOutputs/`. If they do not exist, **AGENT MUST** ask the user to create them following the `Scaffolding Test Project for each Workflow` section above.

## CRITICAL: ActionMock Constructors
**AGENT MUST use** the SDK's **separate constructors** for success and failure scenarios:

```csharp
// For SUCCESSFUL actions - use outputs parameter
new ActionMock(
    status: TestWorkflowStatus.Succeeded,
    name: "Action_Name",
    outputs: mockOutputInstance);

// For FAILED actions - use error parameter (NOT outputs!)
new ActionMock(
    status: TestWorkflowStatus.Failed,
    name: "Action_Name",
    error: new TestErrorInfo(
        code: ErrorResponseCode.ArtifactDownloadFailed,
        message: "Error description",
        details: null,
        additionalInfo: null));
```

**DO NOT** pass outputs to a failed action mock - the SDK validates this and will throw an error.

## CRITICAL: ServiceProvider Output Format
**AGENT MUST ensure** that for `ServiceProvider` type actions (Blob Storage, Service Bus, etc.), outputs have **exactly** these properties:

```csharp
public class MyActionOutput : MockOutput
{
    [JsonProperty("body")]
    public JToken Body { get; set; } = new JObject();

    [JsonProperty("statusCode")]
    public int StatusCode { get; set; } = 200;

    [JsonProperty("headers")]
    public JObject Headers { get; set; } = new JObject();
}
```

**Unexpected properties like `content`, `properties`, etc. will cause validation errors.**

## CRITICAL: Trigger Body Wrapper
**AGENT MUST ensure** that for `@triggerBody()` expressions to work, trigger outputs wrap data in a `body` property:

```csharp
public class MyTriggerOutput : MockOutput
{
    [JsonProperty("body")]
    public JObject Body { get; set; } = new JObject
    {
        ["contentData"] = new JObject { ["key"] = "value" },
        ["properties"] = new JObject(),
        ["messageId"] = Guid.NewGuid().ToString()
    };

    [JsonProperty("statusCode")]
    public int StatusCode { get; set; } = 200;
}
```

## Required Using Statements
```csharp
using Microsoft.Azure.Workflows.UnitTesting;
using Microsoft.Azure.Workflows.UnitTesting.Definitions;
using Microsoft.Azure.Workflows.UnitTesting.ErrorResponses;
using Microsoft.Azure.Workflows.Common.ErrorResponses; // For ErrorResponseCode enum
using Microsoft.VisualStudio.TestTools.UnitTesting;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
```

## Extension Scaffolding Pattern (preferred structure)
```
Tests/
 Tests.sln
 LogicApps/
     LogicApps.Tests.csproj
     TestExecutor.cs              # Factory class with Create() method
     <workflow-name>/
         testSettings.config       # XML config with paths
         MockOutputs/              # Typed mock output classes
            <TriggerName>TriggerOutput.cs
            <TriggerName>TriggerMock.cs
            <ActionName>ActionOutput.cs
            <ActionName>ActionMock.cs
         TC01_<Scenario>/          # Per-test subfolders
             TC01_<Scenario>.cs
```

## Test Organization: Single File vs Multiple Files

**AGENT MUST decide** between two test organization approaches based on project characteristics:

### Option A: Single File with Multiple Test Methods
All test cases in one file (e.g., `<workflow-name>Tests.cs`) with `[ClassInitialize]` for shared setup.

**Advantages:**
- Shared executor initialization - faster test suite execution
- Less code duplication - MockOutput classes reused easily
- Easier refactoring - change common patterns in one place
- Better overview - all tests for one workflow in single file

**When to use:**
- ✅ Number of test cases ≤ 15
- ✅ Tests share many common mocks
- ✅ Single developer or small team
- ✅ Simple to moderate test complexity

### Option B: One File Per Test Case (RECOMMENDED for Workflows)
Each test case in separate file in its own subfolder (e.g., `TC01_<Scenario>/TC01_<Scenario>.cs`).

**Advantages:**
- Complete isolation between tests - no shared state risks
- Easier parallel development - no merge conflicts
- File name directly maps to test case
- Self-contained - each test includes all its mocks and setup
- Matches Extension scaffolding output

**When to use:**
- ✅ Complex workflows with many actions requiring mocks
- ✅ Large team with multiple developers on same workflow
- ✅ Each test requires unique mock configurations
- ✅ CI/CD requires test-level parallelism across processes

### Decision Tree:
```
Workflow has > 5 actions requiring mocks?
├── YES → Use Option B (multiple files) ✓
│         Each test's mock setup is complex enough to warrant isolation
└── NO  → Either option works, consider team size

Number of test cases > 15?
├── YES → Use Option B (multiple files) ✓
└── NO  → Consider Option A for simplicity

Team size > 3 developers on same tests?
├── YES → Use Option B (reduce merge conflicts) ✓
└── NO  → Either option acceptable

═══════════════════════════════════════════════════════════════
DEFAULT FOR WORKFLOWS: Option B (one file per test case)
═══════════════════════════════════════════════════════════════

Reason: Workflow tests typically require complex mock setups,
        and isolation prevents shared state issues.

⚠️ AGENT BEHAVIOR WHEN DEVIATING FROM DEFAULT:
If agent determines Option A (single file) is more appropriate
based on the decision tree above, AGENT MUST:
1. Explain the assessment and reasoning
2. ASK THE USER: "Based on my analysis, I recommend using
   Option A (single file) for this workflow because [reasons].
   Do you agree with this approach, or would you prefer the
   default Option B (one file per test)?"
3. Wait for user confirmation before proceeding
4. If user disagrees, use the default Option B

Default for Data Maps: Use Option A (single file)
Reason: XSLT generation in [ClassInitialize] provides
        significant performance benefit when shared.
```

## TestExecutor Pattern (4-parameter constructor)
```csharp
public class TestExecutor
{
    private readonly string workspacePath;
    private readonly string logicAppName;
    private readonly string workflowName;

    public TestExecutor(string configPath)
    {
        // Load configuration from testSettings.config XML
        var config = XDocument.Load(configPath);
        this.workspacePath = config.Root.Element("WorkspacePath")?.Value 
            ?? throw new InvalidOperationException("WorkspacePath not found in config");
        this.logicAppName = config.Root.Element("LogicAppName")?.Value 
            ?? throw new InvalidOperationException("LogicAppName not found in config");
        this.workflowName = config.Root.Element("WorkflowName")?.Value 
            ?? throw new InvalidOperationException("WorkflowName not found in config");
    }

    public UnitTestExecutor Create()
    {
        string workflowFilePath = Path.Combine(workspacePath, logicAppName, workflowName, "workflow.json");
        string connectionsFilePath = Path.Combine(workspacePath, logicAppName, "connections.json");
        string localSettingsFilePath = Path.Combine(workspacePath, logicAppName, "local.settings.json");
        
        // UnitTestExecutor requires 4 parameters - use null for parametersFilePath if not needed
        return new UnitTestExecutor(workflowFilePath, connectionsFilePath, null, localSettingsFilePath);
    }
}
```

**Example testSettings.config XML structure:**
```xml
<?xml version="1.0" encoding="utf-8"?>
<TestSettings>
  <WorkspacePath>C:\dev\logicapps-project</WorkspacePath>
  <LogicAppName>LogicApps</LogicAppName>
  <WorkflowName>la-process-message</WorkflowName>
</TestSettings>
```

**Required using statement:** Add `using System.Xml.Linq;` to the TestExecutor.cs file.

## Typed Mock Classes Pattern

### Action Mock with Separate Success/Failure Constructors
```csharp
public class ReadBlobContentActionMock : ActionMock
{
    public static readonly string ActionName = "Read_blob_content";

    // Success constructor - uses outputs
    public ReadBlobContentActionMock(MockOutput outputs)
        : base(status: TestWorkflowStatus.Succeeded, name: ActionName, outputs: outputs)
    { }

    // Failure constructor - uses error (NOT outputs)
    public ReadBlobContentActionMock(ErrorResponseCode errorCode, string errorMessage)
        : base(status: TestWorkflowStatus.Failed, name: ActionName, 
               error: new TestErrorInfo(errorCode, errorMessage, null, null))
    { }
}
```

### Trigger Mock
```csharp
public class MyTriggerMock : TriggerMock
{
    public static readonly string TriggerName = "When_a_message_is_received";

    public MyTriggerMock(TestWorkflowStatus status, MockOutput outputs)
        : base(status: status, name: TriggerName, outputs: outputs)
    { }
}
```

## Test Method Pattern
```csharp
[TestMethod]
public async Task TC01_HappyPath_WorkflowSucceeds()
{
    // PREPARE - Create trigger mock with body wrapper
    var triggerOutput = new MyTriggerOutput
    {
        Body = new JObject
        {
            ["contentData"] = new JObject { ["key"] = "value" }
        }
    };
    var triggerMock = new MyTriggerMock(TestWorkflowStatus.Succeeded, triggerOutput);

    // Create action mocks - use success constructor for happy path
    var actionOutput = new MyActionOutput { Body = "response content" };
    var actionMock = new MyActionMock(outputs: actionOutput);

    var actionMocks = new Dictionary<string, ActionMock>
    {
        { MyActionMock.ActionName, actionMock }
    };

    var testMock = new TestMockDefinition(triggerMock, actionMocks);

    // ACT
    var result = await this.testExecutor.Create().RunWorkflowAsync(testMock);

    // ASSERT - Use result.Status and result.Actions (not WorkflowStatus/ActionResults)
    Assert.AreEqual(TestWorkflowStatus.Succeeded, result.Status);
    Assert.AreEqual(TestWorkflowStatus.Succeeded, result.Actions["Action_Name"].Status);
}

[TestMethod]
public async Task TC02_ActionFailure_WorkflowFails()
{
    // PREPARE
    var triggerMock = new MyTriggerMock(TestWorkflowStatus.Succeeded, new MyTriggerOutput());
    
    // Use FAILURE constructor for failed action mock
    var actionMock = new MyActionMock(
        errorCode: ErrorResponseCode.ArtifactDownloadFailed,
        errorMessage: "The resource was not found");

    var actionMocks = new Dictionary<string, ActionMock>
    {
        { MyActionMock.ActionName, actionMock }
    };

    var testMock = new TestMockDefinition(triggerMock, actionMocks);

    // ACT
    var result = await this.testExecutor.Create().RunWorkflowAsync(testMock);

    // ASSERT
    Assert.AreEqual(TestWorkflowStatus.Failed, result.Status);
    Assert.AreEqual(TestWorkflowStatus.Failed, result.Actions["Action_Name"].Status);
}
```

## Common ErrorResponseCode Values
| Code | Use Case |
|------|----------|
| `ArtifactDownloadFailed` | Resource not found (404-like) |
| `InvalidRequest` | Bad request / validation error |
| `StorageConnectivityFailed` | Storage connection issues |
| `ServiceProviderActionFailed` | Generic service provider failure |

## Responsibilities
**AGENT MUST follow this execution sequence:**

1. **Verify Pre-Execution Checklist** (see section above)
2. **Create/Extend Test Classes**:
   - Create MSTest classes with `[TestClass]` attribute
   - Add `[TestMethod]` methods following naming convention: `TC##_<Scenario>_<ExpectedOutcome>`
3. **For Each Test Method**:
   - Initialize the SDK executor using `TestExecutor.Create()` pattern
   - Provide trigger mocks with **body wrapper** for @triggerBody() to work
   - Use **separate constructors** for success vs failure action mocks
   - Execute workflow via `RunWorkflowAsync(testMock)`
   - Assert using `result.Status` and `result.Actions["ActionName"].Status`
4. **Handle Missing Infrastructure**:
   - If test project doesn't exist, **STOP** and request user to scaffold
   - If MockOutput classes missing, **STOP** and request user to scaffold

## Post-Implementation Validation
**AGENT SHOULD perform these checks after creating tests:**

1. ✅ Verify all test files are created in correct locations:
   - Test classes in `Tests/LogicApps/<workflow-name>/TC##_<Scenario>/`
   - MockOutput classes in `Tests/LogicApps/<workflow-name>/MockOutputs/`
2. ✅ Confirm all required using statements are present
3. ✅ Check that file naming follows conventions:
   - Test files: `TC##_<Scenario>.cs`
   - Mock files: `<ActionName>Mock.cs`, `<ActionName>Output.cs`
4. ✅ Validate no compilation errors exist (use get_errors tool)
5. ✅ Suggest running tests: `dotnet test` from Tests/LogicApps directory

## Output Deliverables
**AGENT MUST provide:**

1. **Test Class Files** (`.cs`):
   - Location: `Tests/LogicApps/<workflow-name>/TC##_<Scenario>/TC##_<Scenario>.cs`
   - Contains: `[TestClass]` with one or more `[TestMethod]` async Task methods
   - Naming: Follow pattern `TC##_<ScenarioName>_<ExpectedOutcome>()`

2. **MockOutput Classes** (`.cs`) - if created:
   - Location: `Tests/LogicApps/<workflow-name>/MockOutputs/`
   - Contains: Classes inheriting from `MockOutput` with `[JsonProperty]` attributes
   - Naming: `<TriggerName>Output.cs`, `<ActionName>Output.cs`

3. **Mock Wrapper Classes** (`.cs`) - if created:
   - Location: `Tests/LogicApps/<workflow-name>/MockOutputs/`
   - Contains: Classes inheriting from `TriggerMock` or `ActionMock`
   - Naming: `<TriggerName>Mock.cs`, `<ActionName>Mock.cs`

4. **Documentation Comments**:
   - Add `// TODO:` comments where human input is required (e.g., actual payload values)
   - Include inline comments explaining complex mock setups

5. **Summary Report**:
   - List of files created/modified
   - Any outstanding TODOs or user actions required
   - Compilation status (if verified)
