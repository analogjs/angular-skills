# angular-microfrontend

A skill for building microfrontend architectures with Angular (v20+). Covers runtime integration patterns, bundling strategies, exposing and consuming remotes, and recommended practices for shared libraries, routing, and deployment.

## Summary

- Focus: architecting Angular apps as composable microfrontends (MFEs) with strong isolation, lightweight runtime composition, and safe dependency sharing.
- Approaches: Module Federation (Webpack), import maps / dynamic imports, Web Components (Angular custom elements), and runtime composition via import maps or a shell loader.
- Targets: route-level remotes, widget remotes, and remote components loaded on demand.

## Key Patterns

- Shell + Remotes: A thin shell app handles global concerns (auth, layout, navigation); remotes provide features as lazy-loaded bundles.
- Shared dependencies: mark Angular, RxJS, and common libs as shared singletons to avoid duplication and inconsistent injectors.
- Expose standalone components: prefer standalone components or exported functions that mount into a host node.
- Web Components: compile remotes as custom elements when cross-framework compatibility is required.
- Version negotiation: use strict semantic-versioning for shared libs and fallback strategies for incompatibilities.

## Example: Expose a standalone component with Module Federation

webpack.config.js (remote)

```js
// expose: './UserWidget': './src/app/user-widget/user-widget.component.ts'
module.exports = {
  // Module Federation config (simplified)
  plugins: [
    new ModuleFederationPlugin({
      name: 'user_remote',
      filename: 'remoteEntry.js',
      exposes: {
        './UserWidget': './src/app/user-widget/user-widget.component.ts',
      },
      shared: { '@angular/core': {singleton:true}, '@angular/common': {singleton:true}, rxjs: {singleton:true} }
    })
  ]
}
```

Consumer (shell) loads at runtime:

```ts
// dynamically load remote entry (loadRemoteEntry helper from @module-federation/utilities)
await loadRemoteEntry('http://localhost:4201/remoteEntry.js', 'user_remote');
const { UserWidget } = await window.user_remote.get('./UserWidget');
// mount or render component (if standalone) into host
const el = document.createElement('div');
document.querySelector('#host')!.appendChild(el);
bootstrapStandalone(UserWidget, { host: el });
```

## Example: Build a remote as a custom element

In the remote app, define and register element:

```ts
const injector = createInjector();
const el = createCustomElement(UserWidgetComponent, { injector });
customElements.define('user-widget', el);
```

Then the host can simply add `<user-widget user-id="42"></user-widget>` to the DOM.

## Runtime composition strategies

- Import maps / dynamic imports: good for simple shells that load ES modules by specifier.
- Module Federation: powerful for shared dependency negotiation and code splitting.
- Web Components: useful for cross-framework or independent deployment.

## Recommendations

- Keep the host minimal; push feature logic to remotes.
- Share only stable, widely-used libraries; keep other libs bundled to avoid version conflicts.
- Use CI to validate remote compatibility with the shell (smoke-tests + contract tests).
- Prefer standalone components and small public APIs for remotes.

## See also

- `references/microfrontend-patterns.md` for deeper patterns and deployment notes.
