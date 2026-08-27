# Frontend technical specification

## Framework and structure

- Use Vue.js at the version declared in `AGENTS.md`.
- Prefer the Vue 3 Composition API unless an established repository standard requires otherwise.
- Build the web UI as a static single-page application unless the feature specification requires another rendering model.
- Keep components small, composable, semantic, accessible, and testable.
- Implement only routes, components, forms, tables, navigation, and behavior defined by the feature specification.

## State and interaction

- Keep state local unless multiple areas genuinely require shared state.
- Keep state transitions and data flow explicit, simple, and predictable.
- Represent loading, empty, success, error, and submitting states where applicable.
- Disable duplicate submissions while an asynchronous mutation is pending.
- Handle request failures and appropriate retries gracefully.
- Require confirmation before destructive operations when specified by the feature.
- Validate forms for user experience, but never rely on client validation for security or correctness.

## API integration

- Access backend APIs through a dedicated client layer.
- Centralize endpoint calls and use request and response models from `shared/`.
- Do not duplicate endpoint or transport logic in components.
- Keep provider and deployment details out of UI code.

## Quality

- Maintain accessibility and semantic markup, including programmatically identifiable error messages.
- Avoid unnecessary dependencies.
- Test every user-visible behavior and all applicable UI states according to `testing.md`.
