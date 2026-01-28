---
name: la-unit-tests-create-cases
description: Create unit test cases from a Logic Apps Standard workflow definition, describing scenarios and mapping them to mocks and expected outcomes.
---

## Responsibilities
- Propose a set of test scenarios for the workflow:
  - At minimum: one success scenario; and one failure/error-handling scenario where applicable.
- For each test case, define:
  - Scenario intent
  - Inputs
  - Mock plan (trigger + action mocks with TestWorkflowStatus)
  - Expected workflow outcome (status/outputs/errors/action outcomes)

## Standard Test Case Categories
| Category | Description | Workflow Status |
|----------|-------------|----------------|
| Happy Path | All actions succeed | Succeeded |
| Input Validation | Empty/invalid inputs | Succeeded or Failed |
| Action Failure | External dependency fails | Failed |
| Partial Failure | Some actions fail, others skip | Failed |
| Edge Cases | Large payloads, special characters | Succeeded |
| Data Verification | Verify transformations/outputs | Succeeded |

## Mock Status Values (TestWorkflowStatus enum)
- `Succeeded` - Action completed successfully
- `Failed` - Action failed (requires TestErrorInfo)
- `Skipped` - Action was skipped (due to runAfter failure)
- `TimedOut` - Action timed out

## Naming Convention
Format: `TC<##>_<BriefScenarioName>` (e.g., `TC01_SuccessfulEndToEndFlow`, `TC03_BlobUploadFailure`)

## Test Case Template
```markdown
### TC<##>: <Scenario Name>
- **Intent**: What this test validates
- **Preconditions**: Required setup
- **Mock Plan**:
  - TriggerMock: <trigger_name> → Succeeded
  - ActionMocks:
    - <action_name> → Succeeded/Failed/Skipped
- **Inputs**: Trigger payload description
- **Expected**:
  - Workflow Status: Succeeded/Failed
  - Action Outcomes:
    - <action_name>: Succeeded/Failed/Skipped
  - Outputs: Expected response/data
  - Errors: Expected error (if Failed)
```

## Failure Scenario Requirements
When mocking a Failed action:
- Must provide `TestErrorInfo` with:
  - `ErrorResponseCode` enum value (e.g., `ServiceProviderActionFailed`)
  - Error message string
- Downstream actions with `runAfter` dependency will be `Skipped`

## Output
- A Speckit-style scenario section per test case (see speckit skill)
- Clear mapping between mock names and workflow.json action names
