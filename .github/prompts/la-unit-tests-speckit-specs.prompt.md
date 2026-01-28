---
name: la-unit-tests-speckit-specs
description: Write and maintain Speckit-style specification documents for Logic Apps unit tests so scenarios remain reusable and consistent.
---

## Workspace Setup
Store specs at workspace root: `<workspace>/plan/<workflow-name>-testplan.md` (use `code --add <workspace>/plan` if needed).

## Responsibilities
- For each workflow, create or update a spec document that includes:
  - Workflow overview
  - Test case catalog
  - Per test case: intent, setup, mocks, inputs, expected results
- Keep specs stable as the reusable source, and treat code as implementation.

## Recommended Format

```markdown
# Unit Test Specification: <workflow-name>

## Workflow Overview
| Property | Value |
|----------|-------|
| Name | <workflow-name> |
| Trigger | <trigger_name> (<trigger_type>) |
| Purpose | Brief description |

## Actions Summary
| Action Name | Type | Requires Mock |
|-------------|------|---------------|
| <action_name> | ServiceProvider | Yes |
| <action_name> | Compose | No |

## External Dependencies (Mock Required)
1. **<Action_Name>** - <Connector type> - <Purpose>
2. **<Action_Name>** - <Connector type> - <Purpose>

## Test Case Catalog
| ID | Name | Category | Expected Status |
|----|------|----------|----------------|
| TC01 | SuccessfulEndToEndFlow | Happy Path | Succeeded |
| TC02 | <Scenario> | <Category> | <Status> |

---

## Test Cases

### TC01: Successful End-to-End Flow

**Intent**: Verify the workflow completes successfully when all actions succeed.

**Preconditions**: None

**Mock Plan**:
| Component | Name | Status | Output/Error |
|-----------|------|--------|-------------|
| Trigger | <trigger_name> | Succeeded | Valid payload |
| Action | <action_name> | Succeeded | MockOutput |
| Action | <action_name> | Succeeded | MockOutput |

**Inputs**:
```json
{ "example": "payload" }
```

**Expected Results**:
- Workflow Status: `Succeeded`
- Action Outcomes:
  - <action_name>: `Succeeded`
  - <action_name>: `Succeeded`
- Outputs: <description>

---

### TC<##>: <Failure Scenario>

**Intent**: Verify workflow fails appropriately when <action> fails.

**Mock Plan**:
| Component | Name | Status | Output/Error |
|-----------|------|--------|-------------|
| Trigger | <trigger_name> | Succeeded | Valid payload |
| Action | <action_name> | Failed | ErrorResponseCode.ServiceProviderActionFailed |
| Action | <downstream_action> | Skipped | N/A |

**Expected Results**:
- Workflow Status: `Failed`
- Action Outcomes:
  - <action_name>: `Failed`
  - <downstream_action>: `Skipped`
- Error: <error description>
```

## SDK Reference for Specs
- TestWorkflowStatus: `Succeeded`, `Failed`, `Skipped`, `TimedOut`
- ErrorResponseCode values: `ServiceProviderActionFailed`, `ActionFailed`, etc.
- Mock classes: `TriggerMock`, `ActionMock`, `MockOutput`, `TestErrorInfo`
