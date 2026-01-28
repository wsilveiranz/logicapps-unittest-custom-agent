````prompt
---
name: dm-unit-tests-create-cases
description: Create unit test cases from a Logic Apps Standard data map definition, describing scenarios and mapping them to input data and expected outputs.
---

## Responsibilities
- Propose a set of test scenarios for the data map:
  - At minimum: one success scenario with valid input data
  - Edge case scenarios for boundary conditions
  - Error handling scenarios for invalid input
- For each test case, define:
  - Scenario intent
  - Input data description
  - Expected output structure
  - Validation criteria

## Standard Test Case Categories
| Category | Description | Expected Behavior |
|----------|-------------|-------------------|
| Happy Path | Valid input, all mappings succeed | Correct output produced |
| Empty Input | Empty or minimal input | Handle gracefully, defaults applied |
| Missing Fields | Optional fields not present | Mapped with defaults or omitted |
| Null Values | Null values in source | Handle according to mapping rules |
| Boundary Values | Min/max values for numeric fields | Correct transformation |
| Special Characters | Unicode, XML entities, etc. | Proper encoding/escaping |
| Large Data | Arrays with many items | All items transformed |
| Complex Nesting | Deeply nested structures | Correct hierarchy preservation |
| Date/Time Formats | Various date/time inputs | Correct format conversion |
| Numeric Precision | Decimal calculations | Correct precision maintained |

## Naming Convention
Format: `TC<##>_<BriefScenarioName>` (e.g., `TC01_ValidOrderTransformation`, `TC03_MissingOptionalFields`)

## Test Case Template
```markdown
### TC<##>: <Scenario Name>
- **Intent**: What this test validates
- **Category**: Happy Path / Edge Case / Error Handling
- **Input Data**:
  - Format: XML / JSON
  - Description: Brief description of input characteristics
  - Key fields: List of fields being tested
- **Expected Output**:
  - Format: XML / JSON
  - Key assertions: What to verify in output
  - Specific values: Expected field values
- **Validation Criteria**:
  - [ ] Output structure matches target schema
  - [ ] Field X maps correctly to Field Y
  - [ ] Calculated field Z equals expected value
  - [ ] Date format is correct
```

## Test Case Design Strategy

### 1. Analyze Mapping Rules
For each mapping rule in the data map:
- Identify the source field(s)
- Identify the target field
- Identify any transformation applied
- Create test cases to verify the transformation

### 2. Boundary Analysis
For each field type:
| Field Type | Boundary Tests |
|------------|----------------|
| String | Empty, max length, special chars |
| Number | Zero, negative, max value, decimal precision |
| Date | Min date, max date, various formats |
| Array | Empty, one item, max items |
| Boolean | True, false, missing |

### 3. Transformation Verification
For each transformation type:
| Transformation | Test Cases Needed |
|----------------|-------------------|
| Concatenation | All inputs present, some empty, all empty |
| Substring | Start, end, middle, out of bounds |
| Math | Zero values, negative, overflow |
| Conditional | All branches, default case |
| Loop | 0, 1, many iterations |

## Sample Test Case Catalog
```markdown
## Test Case Catalog for OrderToInvoice Map

| ID | Name | Category | Input Characteristic | Expected Behavior |
|----|------|----------|----------------------|-------------------|
| TC01 | ValidOrderTransformation | Happy Path | Complete order with 3 items | Full invoice generated |
| TC02 | EmptyLineItems | Edge Case | Order with no line items | Invoice with empty items array |
| TC03 | SingleLineItem | Edge Case | Order with 1 line item | Invoice with 1 line item |
| TC04 | MissingOptionalFields | Edge Case | Order without PO number | Invoice with empty PO field |
| TC05 | SpecialCharactersInName | Edge Case | Customer name with XML entities | Properly escaped in output |
| TC06 | MaximumQuantity | Boundary | Item quantity at INT_MAX | Correct total calculation |
| TC07 | NegativePrice | Error Case | Negative unit price | Validation error or handling |
| TC08 | DateFormatConversion | Edge Case | Various date formats | ISO 8601 output format |
| TC09 | UnicodeCharacters | Edge Case | Chinese/Arabic characters | Proper encoding preserved |
| TC10 | LargeOrder | Edge Case | Order with 100+ line items | All items transformed |
```

## Output
- A Speckit-style scenario section per test case (see speckit skill)
- Clear mapping between input fields and expected output fields
- Sample input data structure
- Expected output structure with specific values

````
