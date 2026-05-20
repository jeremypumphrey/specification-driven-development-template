# Frontend instructions

## Framework
- Use Vue.js for the web UI.
- Prefer Vue 3 composition API unless the repository already uses a different established standard.
- Keep components small, composable, and testable.

## UI behavior
- Implement only the UI behavior described in the spec.
- Keep state management simple and explicit.
- Keep forms, tables, navigation, and state flows simple and predictable.
- Validate input on the client for user experience, but do not rely on client-side validation for security or correctness.

## API integration
- Consume the Lambda API through a clear client layer.
- Centralize API calls and shared request/response models.
- Do not duplicate endpoint logic inside components.

## State and data
- Keep state local unless shared state is genuinely needed.
- Represent loading, empty, success, and error states clearly.
- Handle retries and failures gracefully.

## Quality bar
- Add tests for user-visible behavior.
- Maintain accessibility and semantic markup.
- Avoid unnecessary dependencies.