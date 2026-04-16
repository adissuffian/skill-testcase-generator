---
name: msw-testcase-generator
description: "Generate MSW test cases and scenarios with full test coverage from a JIRA ticket. Use when: creating msw tests from JIRA tickets, generating msw test scenarios, writing unit tests from acceptance criteria, test coverage for a ticket."
argument-hint: "Provide a JIRA ticket URL (e.g., https://dominos.atlassian.net/browse/APP-2517) or ticket key (e.g., APP-2517)"
---

# MSW Test Case Generator from JIRA Ticket

Generate comprehensive test cases and scenarios with full coverage from a JIRA ticket's requirements, acceptance criteria, and description.

## When to Use

- When you have a JIRA ticket and need to write test cases before or during implementation
- When you want full test coverage for a feature or bug fix
- When you need test scenarios documented for QA review

## Input

The user provides either:
- A full JIRA URL: `https://dominos.atlassian.net/browse/APP-2517`
- A JIRA ticket key: `APP-2517`

Extract the ticket key from the URL if a full URL is provided (e.g., `APP-2517` from the URL above).

## Procedure

### Step 1: Fetch JIRA Ticket Details

Use the GitKraken MCP tool `mcp_gitkraken_issues_get_detail` to fetch the ticket:
- **provider**: `"jira"`
- **id**: The ticket key (e.g., `APP-2517`)

Extract from the ticket:
- **Title/Summary**
- **Description**
- **Acceptance Criteria** (often in description or a custom field)
- **Type** (Bug, Story, Task, etc.)
- **Labels/Components**
- **Linked issues** (parent epics, sub-tasks, related tickets)

### Step 2: Analyze Requirements

From the ticket details, identify:
1. **Functional requirements** — What the feature/fix should do
2. **Edge cases** — Boundary conditions, error states, empty states
3. **User interactions** — UI events, navigation flows
4. **Data dependencies** — API calls, Redux state, props
5. **Acceptance criteria** — Explicit pass/fail conditions from the ticket
6. **Regression concerns** — What existing behavior must not break

### Step 3: Explore Relevant Codebase

Based on the ticket details, search the codebase for:
- Related components, hooks, or utilities mentioned in the ticket
- Existing test files for the affected area
- Type definitions and interfaces
- GraphQL queries/mutations if backend integration is involved
- Redux state shape if state management is involved

Use `semantic_search`, `grep_search`, and `file_search` to locate relevant files.

### Step 4: Generate MSW Test Cases

Generate test cases following the project's conventions:

#### Test File Structure
```typescript
import { renderHook, act } from '@testing-library/react-hooks'
// or for components:
import { render, fireEvent, waitFor } from '@testing-library/react-native'

// Project-specific utilities
import { createReduxWrapper, initMockReducerData } from '@dominos/ts-tools'

describe('<FeatureName>', () => {
  beforeEach(() => {
    jest.resetModules()
  })

  afterEach(jest.clearAllMocks)

  describe('when <scenario>', () => {
    it('should <expected behavior>', () => {
      // Arrange
      // Act
      // Assert
    })
  })
})
```

#### Naming Conventions
- Test file: `__tests__/<feature-name>.spec.ts` or `__tests__/<feature-name>.spec.tsx`
- Use kebab-case for file names
- Use descriptive `describe`/`it` blocks

#### Coverage Categories

Generate tests for ALL of the following categories:

1. **Happy Path Tests**
   - Normal/expected user flows
   - Successful API responses
   - Valid input handling

2. **Error Handling Tests**
   - API failure scenarios
   - Network errors
   - Invalid input/data

3. **Edge Case Tests**
   - Empty states (no data, empty arrays)
   - Null/undefined values
   - Boundary values (min/max)
   - Loading states

4. **Integration Tests**
   - Component interaction with hooks
   - Redux state changes
   - Navigation flows

5. **Accessibility Tests** (if UI component)
   - Screen reader labels
   - Touch target sizes
   - Color contrast (where testable)

6. **Platform-Specific Tests** (if applicable)
   - iOS vs Android behavior differences
   - Device-specific rendering

#### Mocking Patterns
```typescript
// Mock hooks
jest.mock('@dominos/hooks-and-hocs/customer/use-customer-details', () => ({
  useCustomerDetails: jest.fn().mockReturnValue({ customer })
}))

// Mock GraphQL
jest.mock('@dominos/business/queries', () => ({
  useGetMenuQuery: jest.fn()
}))

// Mock navigation
const mockNavigate = jest.fn()
jest.mock('@react-navigation/native', () => ({
  useNavigation: () => ({ navigate: mockNavigate })
}))
```

### Step 5: Output Format

Present the test cases in two formats:

#### Format A: Test Scenario Table (for documentation/QA)

```markdown
## Test Scenarios for [TICKET-KEY]: [Title]

| # | Category | Scenario | Given | When | Then | Priority |
|---|----------|----------|-------|------|------|----------|
| 1 | Happy Path | User completes action | ... | ... | ... | P0 |
| 2 | Error | API returns 500 | ... | ... | ... | P1 |
| ...| ... | ... | ... | ... | ... | ... |
```

#### Format B: Implementable Test Code

Full test file(s) with all test cases that can be directly saved and run.

## Coverage Targets

Aim to meet or exceed the project's coverage thresholds:
- **Branches**: 65%
- **Functions**: 75%
- **Lines & Statements**: 80%

## Error Handling

- If the JIRA ticket cannot be fetched, ask the user to provide the ticket details manually
- If the codebase area cannot be identified, ask the user to specify the relevant files/components
