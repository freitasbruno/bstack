# Testing Standards
> Hardcoded harness standards. Not modified per project.
> Test plans are defined per feature in the wip file during /plan.

## Philosophy
- Automate everything that can be automated
- Human tests are a last resort, not a default
- Tests are written during /build, not after
- A feature is not complete until its tests pass

## Required for every feature
- Unit tests for all utility functions
- Unit tests for all service functions with mocked dependencies
- Integration tests for all API routes
- Component tests for interactive UI components

## Test runner
Detect from package.json. If none configured, propose setup during /plan.
Preferred: Vitest for unit and integration, Playwright for end-to-end.

## Coverage expectations
- Utilities: 100%
- Services: 100%
- API routes: 100%
- Components: critical interaction paths
- End-to-end: happy path + critical error states

## What requires human verification
Only these cases require a human test:
- Visual appearance and layout
- Third-party integrations that cannot be mocked
- Device or browser-specific behaviour
- User flows that span multiple sessions

## Test plan format in wip file

### Automated
- [ ] [what is being tested] — [test type: unit / integration / e2e]

### Human verification required
- [ ] [specific action] → [expected result]
