````prompt
---
name: dm-unit-tests-generate-test-data
description: Generate input test data files and expected output files for data map unit tests.
---

## Responsibilities
- For a given test case:
  - Generate appropriate input data (XML or JSON) based on the source schema
  - Generate expected output data based on the target schema and mapping rules
  - Cover success and edge case variants
- Ensure data covers boundary conditions and special cases
- Keep test data files organized and named consistently

## Test Data Folder Structure
```
Tests/DataMaps/<map-name>/
└── TestData/
    ├── Input/
    │   ├── TC01_ValidOrder.xml
    │   ├── TC02_EmptyItems.xml
    │   ├── TC03_SpecialCharacters.xml
    │   └── ...
    └── Expected/
        ├── TC01_ValidOrder_Expected.json
        ├── TC02_EmptyItems_Expected.json
        ├── TC03_SpecialCharacters_Expected.json
        └── ...
```

## File Naming Convention
```
Input files:     TC<##>_<ScenarioName>.<ext>
Expected files:  TC<##>_<ScenarioName>_Expected.<ext>

Extensions:
- .xml  - XML format input/output
- .json - JSON format input/output
```

## Input Data Generation Guidelines

### 1. Analyze Schema Structure
Before generating test data:
- Review the source schema (XSD or JSON Schema)
- Identify required vs optional fields
- Identify field types and constraints
- Identify repeating elements/arrays

### 2. Generate Representative Data
```markdown
| Field Type | Test Values to Include |
|------------|------------------------|
| String | Normal text, empty, special chars, max length |
| Number | 0, positive, negative, decimal, max value |
| Date | Valid dates, boundary dates, various formats |
| Array | Empty, single item, multiple items, max items |
| Boolean | true, false |
| Optional | Present, absent |
```

### 3. Input Data Templates

#### XML Input Template
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- Test Case: TC01_ValidOrder -->
<!-- Purpose: Verify complete order transformation -->
<Order xmlns="http://example.com/schemas/order">
  <OrderId>ORD-001</OrderId>
  <OrderDate>2024-01-15T10:30:00Z</OrderDate>
  <Customer>
    <CustomerId>CUST-100</CustomerId>
    <Name>Contoso Corporation</Name>
    <Email>orders@contoso.com</Email>
    <Address>
      <Street>123 Main Street</Street>
      <City>Seattle</City>
      <State>WA</State>
      <PostalCode>98101</PostalCode>
      <Country>USA</Country>
    </Address>
  </Customer>
  <Items>
    <Item>
      <LineNumber>1</LineNumber>
      <ProductId>PROD-A001</ProductId>
      <Description>Premium Widget</Description>
      <Quantity>5</Quantity>
      <UnitPrice>29.99</UnitPrice>
    </Item>
    <Item>
      <LineNumber>2</LineNumber>
      <ProductId>PROD-B002</ProductId>
      <Description>Standard Gadget</Description>
      <Quantity>10</Quantity>
      <UnitPrice>14.50</UnitPrice>
    </Item>
  </Items>
  <Notes>Please expedite shipping</Notes>
</Order>
```

#### JSON Input Template
```json
{
  "_comment": "Test Case: TC01_ValidOrder - Verify complete order transformation",
  "orderId": "ORD-001",
  "orderDate": "2024-01-15T10:30:00Z",
  "customer": {
    "customerId": "CUST-100",
    "name": "Contoso Corporation",
    "email": "orders@contoso.com",
    "address": {
      "street": "123 Main Street",
      "city": "Seattle",
      "state": "WA",
      "postalCode": "98101",
      "country": "USA"
    }
  },
  "items": [
    {
      "lineNumber": 1,
      "productId": "PROD-A001",
      "description": "Premium Widget",
      "quantity": 5,
      "unitPrice": 29.99
    },
    {
      "lineNumber": 2,
      "productId": "PROD-B002",
      "description": "Standard Gadget",
      "quantity": 10,
      "unitPrice": 14.50
    }
  ],
  "notes": "Please expedite shipping"
}
```

## Expected Output Generation Guidelines

### 1. Apply Mapping Rules
For each mapping rule:
- Apply the transformation to the input data
- Calculate any derived fields
- Handle optional fields according to mapping rules

### 2. Expected Output Templates

#### JSON Expected Output (common for DataMapTestExecutor)
```json
{
  "_testCase": "TC01_ValidOrder",
  "invoiceNumber": "ORD-001",
  "invoiceDate": "2024-01-15",
  "billTo": {
    "name": "Contoso Corporation",
    "address": "123 Main Street, Seattle, WA 98101, USA"
  },
  "lineItems": [
    {
      "productCode": "PROD-A001",
      "description": "Premium Widget",
      "quantity": 5,
      "unitPrice": 29.99,
      "lineTotal": 149.95
    },
    {
      "productCode": "PROD-B002",
      "description": "Standard Gadget",
      "quantity": 10,
      "unitPrice": 14.50,
      "lineTotal": 145.00
    }
  ],
  "subtotal": 294.95,
  "tax": 26.55,
  "totalAmount": 321.50
}
```

## Edge Case Test Data Examples

### Empty Array Test
```xml
<!-- TC02_EmptyItems - Order with no line items -->
<Order>
  <OrderId>ORD-002</OrderId>
  <Customer>
    <Name>Empty Order Corp</Name>
  </Customer>
  <Items/>
</Order>
```

### Missing Optional Fields Test
```xml
<!-- TC03_MinimalOrder - Only required fields present -->
<Order>
  <OrderId>ORD-003</OrderId>
  <Customer>
    <Name>Minimal Corp</Name>
  </Customer>
  <Items>
    <Item>
      <ProductId>PROD-X</ProductId>
      <Quantity>1</Quantity>
      <UnitPrice>10.00</UnitPrice>
    </Item>
  </Items>
</Order>
```

### Special Characters Test
```xml
<!-- TC04_SpecialCharacters - XML entities and Unicode -->
<Order>
  <OrderId>ORD-004</OrderId>
  <Customer>
    <Name>O'Brien &amp; Associates "Ltd"</Name>
    <Address>
      <Street>123 Main St &lt;Suite 100&gt;</Street>
      <City>São Paulo</City>
    </Address>
  </Customer>
  <Items>
    <Item>
      <ProductId>PROD-日本語</ProductId>
      <Description>Ñoño's Special — "Best" Item</Description>
      <Quantity>1</Quantity>
      <UnitPrice>99.99</UnitPrice>
    </Item>
  </Items>
</Order>
```

### Boundary Values Test
```xml
<!-- TC05_BoundaryValues - Extreme values -->
<Order>
  <OrderId>ORD-005-MAXIMUM-LENGTH-ID-TESTING-1234567890</OrderId>
  <Customer>
    <Name>A</Name>
  </Customer>
  <Items>
    <Item>
      <ProductId>P</ProductId>
      <Quantity>999999999</Quantity>
      <UnitPrice>0.01</UnitPrice>
    </Item>
    <Item>
      <ProductId>Q</ProductId>
      <Quantity>1</Quantity>
      <UnitPrice>99999999.99</UnitPrice>
    </Item>
  </Items>
</Order>
```

### Large Data Test
```xml
<!-- TC06_LargeOrder - Many line items -->
<Order>
  <OrderId>ORD-006</OrderId>
  <Customer>
    <Name>Bulk Order Inc</Name>
  </Customer>
  <Items>
    <!-- Include 50+ items to test iteration performance -->
    <Item><ProductId>P001</ProductId><Quantity>1</Quantity><UnitPrice>1.00</UnitPrice></Item>
    <Item><ProductId>P002</ProductId><Quantity>2</Quantity><UnitPrice>2.00</UnitPrice></Item>
    <!-- ... continue for 50+ items ... -->
  </Items>
</Order>
```

## Test Data Validation Checklist
Before finalizing test data:
- [ ] Input file is well-formed XML/JSON
- [ ] Input matches source schema structure
- [ ] Expected output matches target schema structure
- [ ] Calculated fields have correct values
- [ ] Edge cases are properly represented
- [ ] File encoding is UTF-8
- [ ] No sensitive/real data included

## Output
- Input test data files in `TestData/Input/`
- Expected output files in `TestData/Expected/`
- Documentation of what each test case validates

````
