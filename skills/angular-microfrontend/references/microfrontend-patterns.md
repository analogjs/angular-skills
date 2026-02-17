# Microfrontend Patterns

Detailed patterns and operational guidance for Angular microfrontends.

## Architecture

- Shell (host): handles global layout, authentication, and routing coordination.
- Remotes: independently deployable apps providing feature areas.
- Communication: use cross-window events, shared state slices exposed via well-defined APIs, or custom events on DOM for Web Components.

## Dependency Sharing

- Mark core framework libs as singletons in Module Federation (`@angular/core`, `@angular/common`, `rxjs`).
- Avoid sharing many third-party libs unless versions are tightly controlled.
- Use semantic versioning and a compatibility matrix in CI to detect breaking changes.

## Routing

- Route-level remotes: the shell delegates certain route trees to remotes. The shell lazy-loads remote entry then mounts remote router or component.
- Path collisions: namespace remote routes to avoid conflicts (e.g., `/users/*` handled by users remote).

## Deployment & CI

- Independent pipelines: each remote builds and deploys independently to its own static host or CDN.
- Contract tests: publish a small contract (public APIs and test harness) for the shell to validate remote compatibility.

## Testing

- Unit test remotes independently.
- End-to-end: run a harness that boots the shell and one or more remotes together.

## Security

- Validate and sanitize inputs across remote boundaries.
- Use CSP and proper subresource integrity (SRI) where possible for remote assets.

## Performance

- Keep remotes small and lazy-load features on demand.
- Use prefetch hints from the shell to warm important remotes.

## When to Use What

- Module Federation: when you need shared runtime negotiation and fine-grained dependency sharing.
- Import maps / ESM: when using native ESM-based hosting, fewer build-time hooks needed.
- Web Components: when cross-framework reuse or isolated DOM is required.
