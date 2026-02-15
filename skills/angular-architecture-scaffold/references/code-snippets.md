# Snippets para scaffold Angular (signals, zoneless, standalone)

Usar al generar archivos base. Respetar el documento de arquitectura si contradice algo aquí.

---

## main.ts (zoneless)

```typescript
import { bootstrapApplication } from '@angular/platform-browser';
import { appConfig } from './app/app.config';
import { AppComponent } from './app/app';

bootstrapApplication(AppComponent, appConfig);
```

---

## app.config.ts (zoneless)

```typescript
import { ApplicationConfig, provideZonelessChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient, withFetch } from '@angular/common/http';
import { routes } from './routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideZonelessChangeDetection(),
    provideRouter(routes),
    provideHttpClient(withFetch()),
  ],
};
```

Si el documento no exige zoneless, usar `provideZoneChangeDetection()` en lugar de `provideZonelessChangeDetection()`.

---

## routes.ts

```typescript
import { Routes } from '@angular/router';

export const routes: Routes = [
  { path: '', pathMatch: 'full', redirectTo: 'home' },
  {
    path: 'home',
    loadComponent: () =>
      import('./features/home/home').then((m) => m.HomeComponent),
  },
  { path: '**', redirectTo: 'home' },
];
```

---

## Componente feature principal (ej. features/home/home.ts)

Sin sufijo en el nombre del archivo. Standalone, OnPush, signals, inject.

```typescript
import { Component, ChangeDetectionStrategy } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-home',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [CommonModule],
  template: `<p>Home works.</p>`,
})
export class HomeComponent {}
```

Con input/output e estado interno (ejemplo):

```typescript
import { Component, ChangeDetectionStrategy, input, output, signal, computed, inject } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-user-profile',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [CommonModule],
  template: `<p>{{ displayName() }}</p>`,
})
export class UserProfileComponent {
  private readonly userService = inject(UserService);

  readonly userId = input.required<string>();
  readonly userSaved = output<User>();

  private readonly _loading = signal(false);
  readonly loading = this._loading.asReadonly();
  readonly user = signal<User | null>(null);

  protected readonly displayName = computed(() => this.user()?.name ?? 'Guest');
}
```

---

## Servicio (ej. core/services/api.ts)

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable({ providedIn: 'root' })
export class ApiService {
  private readonly http = inject(HttpClient);
}
```

---

## Guard funcional (ej. core/guards/auth.ts)

Si el documento pide guards funcionales:

```typescript
import { inject } from '@angular/core';
import { Router, CanActivateFn } from '@angular/router';

export const authGuard: CanActivateFn = () => {
  const router = inject(Router);
  // TODO: inject AuthService and check user
  return true; // or router.createUrlTree(['/login'])
};
```

---

## Modelo (ej. features/products/models/product.ts)

Sin sufijo `.model`. Carpeta `models/` indica el tipo.

```typescript
export interface Product {
  id: string;
  name: string;
  price: number;
}
```
