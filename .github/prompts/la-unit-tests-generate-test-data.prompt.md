---
name: la-unit-tests-generate-test-data
description: Generate trigger/action mock payloads and configurations for a single test case, a workflow, or an entire project.
---

## Responsibilities
- For a given test case:
  - Provide a mock trigger payload.
  - Provide mock action outputs for each external dependency action in the execution path.
- Ensure data covers success and failure variants where relevant.
- Keep mock names meaningful and map mock data to the workflow action names.

## MockOutputs Folder Structure (Extension Pattern)
```
<workflow-name>/
└── MockOutputs/
    ├── <TriggerName>TriggerOutput.cs
    ├── <ActionName>ActionOutput.cs
    └── <ActionName>ActionMock.cs
```

## Typed Mock Output Class Pattern
```csharp
namespace LogicApps.Tests.Mocks.<workflow_name>
{
    /// <summary>
    /// Mock output for <ActionName> action.
    /// </summary>
    public class <ActionName>ActionOutput : MockOutput
    {
        public int StatusCode { get; set; } = 200;
        public JObject Body { get; set; } = new JObject();
        
        // Add action-specific properties based on connector output schema
        public Properties Properties { get; set; } = new Properties();
    }

    public class Properties
    {
        public string BlobName { get; set; }
        public string ContainerName { get; set; }
        // ... connector-specific properties
    }
}
```

## Typed Action Mock Class Pattern
```csharp
namespace LogicApps.Tests.Mocks.<workflow_name>
{
    /// <summary>
    /// Mock for <ActionName> action.
    /// </summary>
    public class <ActionName>ActionMock : ActionMock
    {
        public static readonly string ActionName = "<Exact_Action_Name_From_Workflow>";

        public <ActionName>ActionMock(
            TestWorkflowStatus status = TestWorkflowStatus.Succeeded,
            string name = null,
            MockOutput outputs = null,
            TestErrorInfo error = null)
            : base(
                status: status,
                name: name ?? ActionName,
                outputs: outputs ?? new <ActionName>ActionOutput(),
                error: error)
        { }
    }
}
```

## Common Trigger Mock Patterns

### HTTP Request Trigger
```csharp
public class WhenAnHTTPRequestIsReceivedTriggerOutput : MockOutput
{
    public int StatusCode { get; set; } = 200;
    public JObject Body { get; set; } = new JObject();
    public JObject Headers { get; set; } = new JObject();
}
```

### Service Bus Trigger
```csharp
public class WhenMessagesAreAvailableTriggerOutput : MockOutput
{
    public string ContentData { get; set; }
    public JObject Properties { get; set; } = new JObject();
}
```

## Common Action Mock Patterns

### Blob Storage Action
```csharp
public class UploadBlobActionOutput : MockOutput
{
    public Properties Properties { get; set; } = new Properties();
}

public class Properties
{
    public string Name { get; set; } = "blob-name.txt";
    public string Path { get; set; } = "/container/blob-name.txt";
}
```

### Service Bus Send Action
```csharp
public class SendMessageActionOutput : MockOutput
{
    public string MessageId { get; set; } = Guid.NewGuid().ToString();
    public string SessionId { get; set; }
}
```

## Output
- MockOutputs/ folder with typed output classes
- Mock payload examples mapped to workflow action names
- Test data files (JSON) if file-based approach is used
