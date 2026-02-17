# Angular Skills

A collection of skills for AI-assisted Angular development. These skills provide coding agents such as Claude, Gemini, OpenCode, etc with up-to-date Angular v20+ patterns, best practices, and code examples.

## Installation

Install all skills from this repository:

```bash
npx skills add analogjs/angular-skills
```

Or install individual skills:

```bash
npx skills add analogjs/angular-skills/skills/angular-component
npx skills add analogjs/angular-skills/skills/angular-signals
npx skills add analogjs/angular-skills/skills/angular-forms
# etc.
```

Give the GitHub repo a ⭐️!

## Available Skills

| Skill | Description |
|-------|-------------|
| [angular-component](skills/angular-component) | Standalone components with signal inputs/outputs, OnPush change detection, host bindings, content projection, and lifecycle hooks |
| [angular-di](skills/angular-di) | Dependency injection with `inject()`, injection tokens, provider configuration, and hierarchical DI patterns |
| [angular-directives](skills/angular-directives) | Attribute directives, structural directives for portals/overlays, and host directive composition |
| [angular-forms](skills/angular-forms) | Signal Forms (experimental) with schema-based validation, field state management, and reactive patterns |
| [angular-http](skills/angular-http) | Data fetching with `httpResource()`, `resource()`, HttpClient, and functional interceptors |
| [angular-routing](skills/angular-routing) | Routing with lazy loading, functional guards/resolvers, and signal-based route parameters |
| [angular-signals](skills/angular-signals) | Reactive state with `signal()`, `computed()`, `linkedSignal()`, `effect()`, and RxJS interop |
| [angular-microfrontend](skills/angular-microfrontend) | Microfrontend architectures: Module Federation, Web Components, import maps, and runtime composition |
| [angular-ssr](skills/angular-ssr) | Server-side rendering, incremental hydration, prerendering, and browser-only code patterns |
| [angular-testing](skills/angular-testing) | Testing with TestBed, component harnesses, signal testing, and OnPush component testing |
| [angular-tooling](skills/angular-tooling) | Angular CLI commands, code generation, build configuration, and workspace setup |

## Skill Structure

Each skill follows the standard structure:

```
skills/
└── angular-{name}/
    ├── SKILL.md              # Main skill file with patterns and examples
    └── references/
        └── {name}-patterns.md  # Advanced patterns and additional examples
```

## Angular Version

These skills target **Angular v20+** with modern defaults:

- Standalone components (no `standalone: true` needed)
- Signal-based inputs, outputs, and queries
- Native control flow (`@if`, `@for`, `@switch`)
- `inject()` function for dependency injection
- Functional guards and interceptors
- Signal Forms (experimental)

## Key Patterns

### Components

```typescript
@Component({
  selector: 'app-user-card',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <h2>{{ name() }}</h2>
    @if (showEmail()) {
      <p>{{ email() }}</p>
    }
  `,
})
export class UserCardComponent {
  name = input.required<string>();
  email = input<string>('');
  showEmail = input(false);
  selected = output<string>();
}
```

### Signals

```typescript
const count = signal(0);
const doubled = computed(() => count() * 2);

effect(() => {
  console.log('Count changed:', count());
});
```

### Dependency Injection

```typescript
@Injectable({ providedIn: 'root' })
export class UserService {
  private http = inject(HttpClient);
  private config = inject(APP_CONFIG);
}
```

### HTTP Resources

```typescript
userResource = httpResource<User>(() => `/api/users/${this.userId()}`);
```

### Signal Forms

```typescript
loginModel = signal({ email: '', password: '' });
loginForm = form(this.loginModel, (schemaPath) => {
  required(schemaPath.email);
  email(schemaPath.email);
  required(schemaPath.password);
});
```

## Contributing

Contributions are welcome! Please ensure any additions:

1. Target Angular v20+ patterns
2. Follow the existing skill structure
3. Include practical, working code examples
4. Avoid deprecated patterns (NgModules, `@Input()` decorators, `*ngIf`, etc.)

## Resources

- [Angular Documentation](https://angular.dev)
- [Angular AI Documentation](https://angular.dev/ai)
- [Skills.sh](https://skills.sh)

## License

MIT
