---
name: angular-di
description: Implement dependency injection in Angular v20+ using inject(), injection tokens, and provider configuration. Use for service architecture, providing dependencies at different levels, creating injectable tokens, and managing singleton vs scoped services. Triggers on service creation, configuring providers, using injection tokens, or understanding DI hierarchy.
---

# Angular Dependency Injection

Configure and use dependency injection in Angular v20+ with `inject()` and providers.

## Basic Injection

### Using inject()

Prefer `inject()` over constructor injection:

```typescript
import { Component, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { User } from './user.service';

@Component({
  selector: 'app-user-list',
  template: `...`,
})
export class UserList {
  // Inject dependencies
  private http = inject(HttpClient);
  private userService = inject(User);
  
  // Can use immediately
  users = this.userService.getUsers();
}
```

### Injectable Services

```typescript
import { Injectable, inject, signal } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable({
  providedIn: 'root', // Singleton at root level
})
export class User {
  private http = inject(HttpClient);
  
  private users = signal<User[]>([]);
  readonly users$ = this.users.asReadonly();
  
  async loadUsers() {
    const users = await firstValueFrom(
      this.http.get<User[]>('/api/users')
    );
    this.users.set(users);
  }
}
```

## Provider Scopes

### Choosing the Right Scope

| Scope | When to Use | Lifecycle |
|---|---|---|
| `providedIn: 'root'` | Shared app-wide state, HTTP, auth, logging | Singleton for app lifetime |
| Component `providers` | State isolated to a single component tree (e.g. form wizard, editor) | Created/destroyed with component |
| Route `providers` | State shared within a route subtree but not globally (e.g. admin section) | Created/destroyed with route |

### Root Level (Singleton)

```typescript
// Recommended: providedIn
@Injectable({
  providedIn: 'root',
})
export class Auth {}

// Alternative: in app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [
    Auth,
  ],
};
```

### Component Level (Instance per Component)

```typescript
@Component({
  selector: 'app-editor',
  providers: [EditorState], // New instance for each component
  template: `...`,
})
export class Editor {
  private editorState = inject(EditorState);
}
```

### Route Level

```typescript
export const routes: Routes = [
  {
    path: 'admin',
    providers: [Admin], // Shared within this route tree
    children: [
      { path: '', component: AdminDashboard },
      { path: 'users', component: AdminUsers },
    ],
  },
];
```

## Injection Tokens

### Creating Tokens

```typescript
import { InjectionToken } from '@angular/core';

// Simple value token
export const API_URL = new InjectionToken<string>('API_URL');

// Object token
export interface AppConfig {
  apiUrl: string;
  features: {
    darkMode: boolean;
    analytics: boolean;
  };
}

export const APP_CONFIG = new InjectionToken<AppConfig>('APP_CONFIG');

// Token with factory (self-providing)
export const WINDOW = new InjectionToken<Window>('Window', {
  providedIn: 'root',
  factory: () => window,
});

export const LOCAL_STORAGE = new InjectionToken<Storage>('LocalStorage', {
  providedIn: 'root',
  factory: () => localStorage,
});
```

### Providing Token Values

```typescript
// app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [
    { provide: API_URL, useValue: 'https://api.example.com' },
    {
      provide: APP_CONFIG,
      useValue: {
        apiUrl: 'https://api.example.com',
        features: { darkMode: true, analytics: true },
      },
    },
  ],
};
```

### Injecting Tokens

```typescript
@Injectable({ providedIn: 'root' })
export class Api {
  private apiUrl = inject(API_URL);
  private config = inject(APP_CONFIG);
  private window = inject(WINDOW);
  
  getBaseUrl(): string {
    return this.apiUrl;
  }
}
```

## Provider Types

### useClass

```typescript
// Provide implementation
{ provide: Logger, useClass: ConsoleLogger }

// Conditional implementation
{
  provide: Logger,
  useClass: environment.production
    ? ProductionLogger
    : ConsoleLogger,
}
```

### useValue

```typescript
// Static values
{ provide: API_URL, useValue: 'https://api.example.com' }

// Configuration objects
{ provide: APP_CONFIG, useValue: { theme: 'dark', language: 'en' } }
```

### useFactory

```typescript
// Factory with dependencies
{
  provide: User,
  useFactory: (http: HttpClient, config: AppConfig) => {
    return new User(http, config.apiUrl);
  },
  deps: [HttpClient, APP_CONFIG],
}

// Async factory (not recommended - use provideAppInitializer)
{
  provide: CONFIG,
  useFactory: () => fetch('/config.json').then(r => r.json()),
}
```

### useExisting

```typescript
// Alias to existing provider
{ provide: AbstractLogger, useExisting: ConsoleLogger }

// Multiple tokens pointing to same instance
providers: [
  ConsoleLogger,
  { provide: Logger, useExisting: ConsoleLogger },
  { provide: ErrorLogger, useExisting: ConsoleLogger },
]
```

## Injection Options

### Optional Injection

```typescript
@Component({...})
export class My {
  // Returns null if not provided
  private analytics = inject(Analytics, { optional: true });
  
  trackEvent(name: string) {
    this.analytics?.track(name);
  }
}
```

### Self, SkipSelf, Host

```typescript
@Component({
  providers: [Local],
})
export class Parent {
  // Only look in this component's injector
  private local = inject(Local, { self: true });
}

@Component({...})
export class Child {
  // Skip this component, look in parent
  private parentService = inject(ParentSvc, { skipSelf: true });

  // Only look up to host component
  private hostService = inject(Host, { host: true });
}
```

## Multi Providers

Collect multiple values for same token:

```typescript
// Token for multiple validators
export const VALIDATORS = new InjectionToken<Validator[]>('Validators');

// Provide multiple values
providers: [
  { provide: VALIDATORS, useClass: RequiredValidator, multi: true },
  { provide: VALIDATORS, useClass: EmailValidator, multi: true },
  { provide: VALIDATORS, useClass: MinLengthValidator, multi: true },
]

// Inject as array
@Injectable()
export class Validation {
  private validators = inject(VALIDATORS); // Validator[]
  
  validate(value: string): ValidationError[] {
    return this.validators
      .map(v => v.validate(value))
      .filter(Boolean);
  }
}
```

### HTTP Interceptors (Multi Provider)

```typescript
// Interceptors use multi providers internally
export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([
        authInterceptor,
        loggingInterceptor,
        errorInterceptor,
      ])
    ),
  ],
};
```

## App Initializers

Run async code before app starts using `provideAppInitializer`:

```typescript
import { provideAppInitializer, inject } from '@angular/core';

export const appConfig: ApplicationConfig = {
  providers: [
    Config,
    provideAppInitializer(() => {
      const configService = inject(Config);
      return configService.loadConfig();
    }),
  ],
};
```

### Multiple Initializers

```typescript
providers: [
  provideAppInitializer(() => {
    const config = inject(Config);
    return config.load();
  }),
  provideAppInitializer(() => {
    const auth = inject(Auth);
    return auth.checkSession();
  }),
]
```

## Environment Injector

Create injectors programmatically:

```typescript
import { createEnvironmentInjector, EnvironmentInjector, inject } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class Plugin {
  private parentInjector = inject(EnvironmentInjector);
  
  loadPlugin(providers: Provider[]): EnvironmentInjector {
    return createEnvironmentInjector(providers, this.parentInjector);
  }
}
```

## runInInjectionContext

Run code with injection context:

```typescript
import { runInInjectionContext, EnvironmentInjector, inject } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class Utility {
  private injector = inject(EnvironmentInjector);
  
  executeWithDI<T>(fn: () => T): T {
    return runInInjectionContext(this.injector, fn);
  }
}

// Usage
utilityService.executeWithDI(() => {
  const http = inject(HttpClient);
  // Use http...
});
```

## Validating DI Configuration

Before relying on a newly configured provider, confirm it resolves correctly to catch misconfiguration early.

### Quick console check during development

Log the injected value in the constructor or at field initialisation to confirm the correct instance is resolved:

```typescript
@Injectable({ providedIn: 'root' })
export class Api {
  private apiUrl = inject(API_URL);

  constructor() {
    // Remove after verifying — confirms token is provided and has the expected value
    console.assert(!!this.apiUrl, 'API_URL token is not provided');
    console.log('[Api] apiUrl resolved to:', this.apiUrl);
  }
}
```

### Verify scope with instance identity

For component- or route-scoped services, confirm that separate component trees receive distinct instances:

```typescript
@Component({
  selector: 'app-editor',
  providers: [EditorState],
  template: `...`,
})
export class Editor {
  private editorState = inject(EditorState);

  constructor() {
    // Each Editor instance should log a different object reference
    console.log('[Editor] EditorState instance:', this.editorState);
  }
}
```

### Validate optional tokens

When a token may or may not be provided, assert the expected presence at startup to surface missing providers before they cause silent failures:

```typescript
@Component({...})
export class FeatureComponent {
  private analytics = inject(Analytics, { optional: true });

  constructor() {
    if (!this.analytics) {
      console.warn('Analytics not provided — tracking disabled for this component.');
    }
  }
}
```

## Troubleshooting Common DI Errors

### NullInjectorError: No provider for X

Occurs when a token has no matching provider in the injector hierarchy.

**Diagnose:**
1. Check that the service has `providedIn: 'root'` or is listed in a `providers` array accessible from the injection site.
2. If using a token, confirm the token is provided (e.g. via `useValue`, `useFactory`) in `app.config.ts` or the relevant `providers` array.
3. For component-scoped services, verify the component's own `providers` array includes the service.
4. Use `{ optional: true }` to guard optional dependencies and handle `null` gracefully.

```typescript
// Guard an optional dependency to avoid NullInjectorError at runtime
private analytics = inject(Analytics, { optional: true });
```

### inject() Called Outside Injection Context

`inject()` must be called during class construction or inside `runInInjectionContext`. Calling it in lifecycle hooks or async callbacks throws an error.

```typescript
// ❌ Wrong — inject() called outside construction context
ngOnInit() {
  this.http = inject(HttpClient); // Error
}

// ✅ Correct — inject() at field initialisation time
private http = inject(HttpClient);

// ✅ Correct — inject() inside runInInjectionContext for dynamic use
this.utility.executeWithDI(() => inject(HttpClient));
```

### Circular Dependency

Two or more services that depend on each other directly cause a circular dependency error.

**Diagnose:** Angular's error message will name the cycle. Break it by:
- Extracting shared logic into a third service that neither depends on the other.
- Using `inject()` lazily inside a method rather than at field initialisation (defers resolution).

```typescript
// Extract shared state to break A ↔ B circular dependency
@Injectable({ providedIn: 'root' })
export class SharedState { /* common data */ }

@Injectable({ providedIn: 'root' })
export class A { private shared = inject(SharedState); }

@Injectable({ providedIn: 'root' })
export class B { private shared = inject(SharedState); }
```

For advanced patterns, see [references/di-patterns.md](references/di-patterns.md).
