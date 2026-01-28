````prompt
---
name: dm-unit-tests-implement
description: Implement MSTest unit tests for Logic Apps Standard data maps using the DataMapTestExecutor class from the Automated Test SDK (Microsoft.Azure.Workflows.WebJobs.Tests.Extension) on net8.0.
---

## Pre-Execution Checklist
**AGENT MUST verify these conditions before implementing tests:**

1. ✅ **Data Map Exists**: Confirm `.lml` or `.xslt` file exists in `Artifacts/MapDefinitions/` or `Artifacts/Maps/`
2. ✅ **Test Project Exists**: Check if `Tests/DataMaps/DataMaps.Tests.csproj` exists
3. ✅ **Map Test Folder**: Verify `Tests/DataMaps/<map-name>/` exists for the target map
4. ✅ **Test Data Folders**: Check for `TestData/Input/` and `TestData/Expected/` folders
5. ✅ **Configuration File**: Check for `testSettings.config` in map test folder

**Note:** LML maps and XSLT maps with the same name should share the same test folder (e.g., `Tests/DataMaps/OrderToInvoice/` for both `OrderToInvoice.lml` and `OrderToInvoice.xslt`).

## Critical: Scaffolding the Test Project
**AGENT MUST** verify and scaffold the test project as needed before implementing tests.

### Decision Tree:
```
1. CHECK: Does Tests/DataMaps/DataMaps.Tests.csproj exist?
   
   ✅ YES → Test project exists, continue to step 2
   
   ❌ NO → CHECK: Does Tests/Tests.sln exist?
      
      ✅ Tests.sln exists → SCAFFOLD the DataMaps.Tests.csproj:
         - Create Tests/DataMaps/ folder
         - Create DataMaps.Tests.csproj with required PackageReferences
         - Create DataMapTestExecutorFactory.cs
         - Add project to Tests.sln
         
      ❌ Tests.sln does not exist → STOP and inform user:
         "No test solution found. Please create Tests/Tests.sln first 
          or run 'Create Unit Test' from a Logic Apps workflow to 
          initialize the test project structure."

2. CHECK: Does Tests/DataMaps/<map-name>/ folder exist?
   
   ✅ YES → PROCEED with test implementation
   
   ❌ NO → CREATE the map test folder structure:
      - Tests/DataMaps/<map-name>/
      - Tests/DataMaps/<map-name>/testSettings.config
      - Tests/DataMaps/<map-name>/TestData/Input/
      - Tests/DataMaps/<map-name>/TestData/Expected/
```

### Scaffolding Actions (Agent CAN perform):

**When Tests.sln exists but DataMaps.Tests.csproj does not:**

1. **Create `Tests/DataMaps/` folder**

2. **Create `DataMaps.Tests.csproj`** using the Project File Template below

3. **Create `DataMapTestExecutorFactory.cs`** using the Factory Pattern below

4. **Add to solution**:
   ```powershell
   dotnet sln Tests/Tests.sln add Tests/DataMaps/DataMaps.Tests.csproj
   ```

**When DataMaps.Tests.csproj exists but map folder does not:**

1. **Create map test folder**: `Tests/DataMaps/<map-name>/`

2. **Create testSettings.config** using the template below

3. **Create TestData folders**:
   - `Tests/DataMaps/<map-name>/TestData/Input/`
   - `Tests/DataMaps/<map-name>/TestData/Expected/`

### Map Folder Naming Convention:
- Use the map name without extension as the folder name
- LML and XSLT maps with the same name share the same test folder
- Example: Both `OrderToInvoice.lml` and `OrderToInvoice.xslt` → `Tests/DataMaps/OrderToInvoice/`

## Workspace Setup
Verify these folders exist and are in VS Code workspace (use `code --add <path>` if needed):
- `<workspace>/plan/` - Test specifications
- `<workspace>/Tests/` - Test projects

## Defaults (must enforce)
- Target framework: `net8.0`
- Inline PackageReferences:
  ```xml
    <PackageReference Include="Microsoft.Azure.Workflows.WebJobs.Tests.Extension" Version="1.*" />
    <PackageReference Include="MSTest" Version="4.0.2" />
    <PackageReference Include="coverlet.collector" Version="3.2.0" />
  ```

## MSTest Annotations (required)
**AGENT MUST include** the following annotations on every test class and method:

### Class-Level Annotations:
```csharp
[TestClass]
[TestCategory("DataMap")]            // Always include for data map tests
[TestCategory("<map-name>")]         // e.g., "source_order_to_canonical_order"
public class SourceOrderToCanonicalOrderTests
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
public async Task TC01_ValidOrder_TransformsCorrectly()
{
    // ...
}
```

### Priority Guidelines:
| Priority | Usage |
|----------|-------|
| 1 | Happy path, critical transformation scenarios |
| 2 | Edge cases, optional fields, boundary conditions |
| 3 | Error handling, invalid input scenarios |

### Category Values for TestProperty:
- `HappyPath` - Normal successful transformation paths
- `ErrorHandling` - Invalid input and error scenarios
- `DataVerification` - Field mapping and value validation
- `BoundaryConditions` - Edge cases (empty arrays, null values, limits)

## SDK Core Class: DataMapTestExecutor

### Namespace
```csharp
using Microsoft.Azure.Workflows.UnitTesting;
using Microsoft.Azure.Workflows.Data.Entities;
```

### Constructor
```csharp
// Initialize with Logic Apps project root path
var executor = new DataMapTestExecutor("path/to/logic-app-project");
```

### Key Methods
| Method | Purpose | Path Lookup |
|--------|---------|-------------|
| `GenerateXslt(string mapName)` | Compile LML to XSLT | `{appDirectoryPath}/Artifacts/MapDefinitions/{mapName}.lml` |
| `GenerateXslt(GenerateXsltInput)` | Compile from content | N/A - uses provided content |
| `RunMapAsync(string mapName, byte[] input)` | Execute by XSLT name | `{appDirectoryPath}/Artifacts/Maps/{mapName}.xslt` |
| `RunMapAsync(byte[] xslt, byte[] input)` | Execute with content | N/A - uses provided XSLT |

## Required Using Statements
```csharp
using Microsoft.Azure.Workflows.UnitTesting;
using Microsoft.Azure.Workflows.Data.Entities;
using Microsoft.VisualStudio.TestTools.UnitTesting;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
using System.Text;
```

## Test Project Structure
```
Tests/
├── Tests.sln
└── DataMaps/
    ├── DataMaps.Tests.csproj
    ├── DataMapTestExecutorFactory.cs    # Factory class for executor creation
    └── <map-name>/
        ├── testSettings.config           # XML config with paths
        ├── <map-name>Tests.cs            # Test class file (single file approach)
        └── TestData/
            ├── Input/                     # Input test data files
            │   ├── TC01_ValidOrder.xml
            │   ├── TC02_EmptyItems.xml
            │   └── ...
            └── Expected/                  # Expected output files
                ├── TC01_ValidOrder_Expected.json
                ├── TC02_EmptyItems_Expected.json
                └── ...
```

## Test Organization: Single File vs Multiple Files

**AGENT MUST decide** between two test organization approaches based on project characteristics:

### Option A: Single File with Multiple Test Methods (RECOMMENDED for Data Maps)
All test cases in one file (e.g., `<map-name>Tests.cs`) with `[ClassInitialize]` for shared setup.

**Advantages:**
- XSLT generation happens once via `[ClassInitialize]` - significantly faster execution
- Less code duplication - shared executor and test data path
- Easier refactoring - change common patterns in one place
- Better overview - all tests for one map in single file

**When to use:**
- ✅ Number of test cases ≤ 15
- ✅ Tests share common setup (same map, same executor)
- ✅ Single developer or small team
- ✅ Performance is important (XSLT generation is expensive)

### Option B: One File Per Test Case
Each test case in separate file (e.g., `TC01_ValidOrder/TC01_ValidOrder.cs`).

**Advantages:**
- Complete isolation between tests
- Easier parallel development - no merge conflicts
- File name directly maps to test case
- Smaller diffs when modifying individual tests

**When to use:**
- ✅ Number of test cases > 15
- ✅ Large team with multiple developers on same map
- ✅ Highly complex individual tests requiring extensive setup
- ✅ CI/CD requires test-level parallelism across processes

### Decision Tree:
```
Number of test cases > 15?
├── YES → Consider Option B (multiple files)
│         BUT weigh against XSLT generation overhead
└── NO  → Use Option A (single file) ✓

Team size > 3 developers on same tests?
├── YES → Consider Option B (reduce merge conflicts)
└── NO  → Use Option A (single file) ✓

═══════════════════════════════════════════════════════════════
DEFAULT FOR DATA MAPS: Option A (single file with multiple tests)
═══════════════════════════════════════════════════════════════

Reason: XSLT generation in [ClassInitialize] runs once,
        providing significant performance benefit.

⚠️ AGENT BEHAVIOR WHEN DEVIATING FROM DEFAULT:
If agent determines Option B (one file per test) is more appropriate
based on the decision tree above, AGENT MUST:
1. Explain the assessment and reasoning
2. ASK THE USER: "Based on my analysis, I recommend using
   Option B (one file per test) for this data map because [reasons].
   Do you agree with this approach, or would you prefer the
   default Option A (single file)?"
3. Wait for user confirmation before proceeding
4. If user disagrees, use the default Option A
```

## DataMapTestExecutorFactory Pattern
```csharp
using Microsoft.Azure.Workflows.UnitTesting;
using System.Xml.Linq;

namespace DataMaps.Tests
{
    /// <summary>
    /// Factory class for creating DataMapTestExecutor instances.
    /// </summary>
    public class DataMapTestExecutorFactory
    {
        private readonly string logicAppPath;

        public DataMapTestExecutorFactory(string configPath)
        {
            var config = XDocument.Load(configPath);
            var workspacePath = config.Root.Element("WorkspacePath")?.Value 
                ?? throw new InvalidOperationException("WorkspacePath not found");
            var logicAppName = config.Root.Element("LogicAppName")?.Value 
                ?? throw new InvalidOperationException("LogicAppName not found");
            
            this.logicAppPath = Path.Combine(workspacePath, logicAppName);
        }

        public DataMapTestExecutor Create()
        {
            return new DataMapTestExecutor(logicAppPath);
        }
    }
}
```

## testSettings.config Template
```xml
<?xml version="1.0" encoding="utf-8"?>
<TestSettings>
  <WorkspacePath>../../../..</WorkspacePath>
  <LogicAppName>LogicApps</LogicAppName>
  <MapName>OrderToInvoice</MapName>
</TestSettings>
```

## Test Class Pattern

### Basic Test Class Structure
```csharp
using Microsoft.Azure.Workflows.UnitTesting;
using Microsoft.VisualStudio.TestTools.UnitTesting;
using Newtonsoft.Json.Linq;
using System.Text;

namespace DataMaps.Tests.OrderToInvoice
{
    [TestClass]
    public class OrderToInvoiceTests
    {
        private static DataMapTestExecutor _executor;
        private static string _testDataPath;
        private static byte[] _xslt;

        [ClassInitialize]
        public static async Task ClassInitialize(TestContext context)
        {
            // Get the path to the test folder
            var testFolder = Path.GetDirectoryName(typeof(OrderToInvoiceTests).Assembly.Location);
            var configPath = Path.Combine(testFolder, "OrderToInvoice", "testSettings.config");
            
            // Create executor from factory
            var factory = new DataMapTestExecutorFactory(configPath);
            _executor = factory.Create();
            
            // Pre-generate XSLT for reuse across tests
            _xslt = await _executor.GenerateXslt("OrderToInvoice");
            
            // Set test data path
            _testDataPath = Path.Combine(testFolder, "OrderToInvoice", "TestData");
        }

        [TestMethod]
        public async Task TC01_ValidOrderTransformation_ReturnsCorrectInvoice()
        {
            // ARRANGE - Load input data
            var inputPath = Path.Combine(_testDataPath, "Input", "TC01_ValidOrder.xml");
            var inputData = await File.ReadAllBytesAsync(inputPath);
            
            // ACT - Execute transformation
            var result = await _executor.RunMapAsync(_xslt, inputData);
            
            // ASSERT - Verify output
            Assert.IsNotNull(result);
            
            // Verify specific fields
            Assert.AreEqual("ORD-001", result["InvoiceNumber"]?.ToString());
            Assert.AreEqual(2, result["LineItems"]?.Count());
            Assert.AreEqual(125.00m, result["TotalAmount"]?.Value<decimal>());
        }

        [TestMethod]
        public async Task TC02_EmptyLineItems_ReturnsEmptyItemsArray()
        {
            // ARRANGE
            var inputPath = Path.Combine(_testDataPath, "Input", "TC02_EmptyItems.xml");
            var inputData = await File.ReadAllBytesAsync(inputPath);
            
            // ACT
            var result = await _executor.RunMapAsync(_xslt, inputData);
            
            // ASSERT
            Assert.IsNotNull(result);
            Assert.AreEqual(0, result["LineItems"]?.Count() ?? 0);
        }
    }
}
```

### Alternative: Inline Test Data Pattern
For simpler tests, data can be inline:
```csharp
[TestMethod]
public async Task TC03_SpecialCharacters_ProperlyEscaped()
{
    // ARRANGE - Inline input data
    var inputXml = @"
        <Order>
            <OrderId>ORD-003</OrderId>
            <Customer>
                <Name>O'Brien &amp; Associates</Name>
            </Customer>
            <Items/>
        </Order>";
    var inputData = Encoding.UTF8.GetBytes(inputXml);
    
    // ACT
    var result = await _executor.RunMapAsync(_xslt, inputData);
    
    // ASSERT
    Assert.AreEqual("O'Brien & Associates", result["BillTo"]?["Name"]?.ToString());
}
```

### Test with Expected Output Comparison
```csharp
[TestMethod]
public async Task TC01_ValidOrder_MatchesExpectedOutput()
{
    // ARRANGE
    var inputPath = Path.Combine(_testDataPath, "Input", "TC01_ValidOrder.xml");
    var expectedPath = Path.Combine(_testDataPath, "Expected", "TC01_ValidOrder_Expected.json");
    
    var inputData = await File.ReadAllBytesAsync(inputPath);
    var expectedJson = await File.ReadAllTextAsync(expectedPath);
    var expected = JToken.Parse(expectedJson);
    
    // ACT
    var result = await _executor.RunMapAsync(_xslt, inputData);
    
    // ASSERT - Compare full output
    Assert.IsTrue(JToken.DeepEquals(expected, result), 
        $"Output mismatch.\nExpected: {expected}\nActual: {result}");
}
```

## Assertion Patterns

### Field Value Assertions
```csharp
// String field
Assert.AreEqual("expected", result["FieldName"]?.ToString());

// Numeric field
Assert.AreEqual(100.50m, result["Amount"]?.Value<decimal>());

// Nested field
Assert.AreEqual("value", result["Parent"]?["Child"]?.ToString());

// Array count
Assert.AreEqual(3, result["Items"]?.Count());

// Array item value
Assert.AreEqual("item1", result["Items"]?[0]?["Name"]?.ToString());
```

### Null/Empty Assertions
```csharp
// Field exists but null
Assert.IsNull(result["OptionalField"]?.Value<string>());

// Field missing
Assert.IsFalse(result.ContainsKey("MissingField"));

// Array is empty
Assert.AreEqual(0, result["Items"]?.Count() ?? 0);
```

### Complex Assertions
```csharp
// Verify all items in array
var items = result["Items"] as JArray;
Assert.IsNotNull(items);
foreach (var item in items)
{
    Assert.IsNotNull(item["ProductCode"], "ProductCode should not be null");
    Assert.IsTrue(item["Quantity"]?.Value<int>() > 0, "Quantity should be positive");
}
```

## Error Handling Tests
```csharp
[TestMethod]
public async Task TC_InvalidXml_ThrowsException()
{
    // ARRANGE - Invalid input
    var invalidInput = Encoding.UTF8.GetBytes("not valid xml");
    
    // ACT & ASSERT
    await Assert.ThrowsExceptionAsync<Exception>(async () =>
    {
        await _executor.RunMapAsync(_xslt, invalidInput);
    });
}
```

## Project File Template
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <IsPackable>false</IsPackable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.Azure.Workflows.WebJobs.Tests.Extension" Version="1.*" />
    <PackageReference Include="MSTest" Version="4.0.2" />
  </ItemGroup>

  <ItemGroup>
    <None Update="**\testSettings.config">
      <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    </None>
    <None Update="**\TestData\**\*">
      <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    </None>
  </ItemGroup>
</Project>
```

## ⚠️ CRITICAL CONSTRAINTS SUMMARY
1. **Executor Path**: DataMapTestExecutor requires the Logic Apps project root path, not workflow path
2. **Map Name**: GenerateXslt uses map name without extension (e.g., "OrderToInvoice" not "OrderToInvoice.lml")
3. **Input Format**: Input must be byte[] (use Encoding.UTF8.GetBytes for strings)
4. **Output Format**: RunMapAsync returns JToken, navigate with indexers and null-conditional operators
5. **XSLT Caching**: Generate XSLT once in ClassInitialize for performance
6. **File Paths**: Ensure test data files are copied to output directory

## Responsibilities
**AGENT MUST follow this execution sequence:**

1. **Verify Pre-Execution Checklist** (see section above)
2. **Create/Extend Test Classes**:
   - Create MSTest classes with `[TestClass]` attribute
   - Add `[TestMethod]` methods following naming convention: `TC##_<Scenario>_<ExpectedOutcome>`
3. **For Each Test Method**:
   - Initialize the SDK executor using `DataMapTestExecutorFactory.Create()` pattern
   - Load input test data from `TestData/Input/` folder
   - Execute transformation via `RunMapAsync()`
   - Assert output against expected results
4. **Handle Missing Infrastructure**:
   - If test project doesn't exist, **STOP** and request user to scaffold
   - If test data folders missing, **STOP** and request user to create them

## Post-Implementation Validation
**AGENT SHOULD perform these checks after creating tests:**

1. ✅ Verify all test files are created in correct locations:
   - Test classes in `Tests/DataMaps/<map-name>/`
   - Test data in `Tests/DataMaps/<map-name>/TestData/Input/` and `TestData/Expected/`
2. ✅ Confirm all required using statements are present
3. ✅ Check that file naming follows conventions:
   - Test files: `<map-name>Tests.cs`
   - Input data: `TC##_<Scenario>.xml` or `.json`
   - Expected output: `TC##_<Scenario>_Expected.json`
4. ✅ Validate no compilation errors exist (use get_errors tool)
5. ✅ Suggest running tests: `dotnet test` from Tests/DataMaps directory

## Output Deliverables
**AGENT MUST provide:**

1. **Test Class Files** (`.cs`):
   - Location: `Tests/DataMaps/<map-name>/<map-name>Tests.cs`
   - Contains: `[TestClass]` with one or more `[TestMethod]` async Task methods
   - Naming: Follow pattern `TC##_<ScenarioName>_<ExpectedOutcome>()`

2. **Test Data Files**:
   - Input files: `Tests/DataMaps/<map-name>/TestData/Input/TC##_<Scenario>.xml`
   - Expected files: `Tests/DataMaps/<map-name>/TestData/Expected/TC##_<Scenario>_Expected.json`

3. **Documentation Comments**:
   - Add `// TODO:` comments where human input is required (e.g., actual expected values)
   - Include inline comments explaining complex assertions

4. **Summary Report**:
   - List of files created/modified
   - Any outstanding TODOs or user actions required
   - Compilation status (if verified)
