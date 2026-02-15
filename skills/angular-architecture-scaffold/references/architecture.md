# Referencia: Arquitectura Angular (Scope Rule, standalone, signals, zoneless)

Resumen literal de la arquitectura para scaffold. Fuente: documentos tipo gentleman-programming / context-angular.

---

## 1. Scope Rule (REQUIRED)

**"Scope determines structure"**: la ubicación de un componente depende de su uso.

| Uso | Ubicación |
|-----|-----------|
| Usado por 1 feature | `features/[feature]/components/` |
| Usado por 2+ features | `features/shared/components/` |

Ejemplo:

```
features/
  shopping-cart/
    shopping-cart.ts
    components/
      cart-item.ts
      cart-summary.ts
  checkout/
    checkout.ts
    components/
      payment-form.ts
  shared/
    components/
      button.ts
      modal.ts
```

---

## 2. Estructura del proyecto

```
src/app/
  features/
    [feature-name]/
      [feature-name].ts       # Componente principal (mismo nombre que la carpeta)
      components/
      services/
      models/
    shared/
      components/
      services/
      pipes/
  core/
    services/
    interceptors/
    guards/
  app.ts
  app.config.ts
  routes.ts
  main.ts
```

---

## 3. File Naming (REQUIRED)

Sin sufijos `.component`, `.service`, `.model`. La carpeta indica el tipo.

- `user-profile.ts` (no `user-profile.component.ts`)
- `cart.ts` (no `cart.service.ts`)
- `user.ts` (no `user.model.ts`)

---

## 4. Style guide

- `inject()` sobre inyección por constructor.
- Bindings `class` y `style` sobre `ngClass`/`ngStyle`.
- `protected` para miembros solo usados en template.
- `readonly` para inputs, outputs, queries.
- Handlers nombrados por acción (`saveUser`), no por evento (`handleClick`).
- Un concepto por archivo.

Orden sugerido en la clase:

1. Dependencias inyectadas
2. Inputs/Outputs
3. Estado interno (signals)
4. Computed
5. Métodos

---

## 5. Core: Standalone, inputs/outputs, signals

- Componentes **standalone** por defecto (no poner `standalone: true` explícitamente).
- **Input/Output**: siempre con funciones: `input()`, `input.required()`, `output()`, `model()`.
- **Estado**: signals (`signal()`, `computed()`); actualización con `.set()` / `.update()`.
- **Sin lifecycle hooks**: no usar `ngOnInit`, `ngOnChanges`, `ngOnDestroy`. Usar `effect()` para reaccionar a inputs o side effects; `computed()` para datos derivados; `DestroyRef` para limpieza.
- **Inyección**: `inject()` en lugar de constructor.

---

## 6. Control flow nativo

- `@if` / `@else`
- `@for (item of items(); track item.id)` con `@empty`
- `@switch` / `@case` / `@default`

No usar `*ngIf` / `*ngFor`.

---

## 7. Zoneless

- `provideZonelessChangeDetection()` en `bootstrapApplication`.
- Eliminar `zone.js` del proyecto y de `angular.json` (polyfills).
- Requisitos: OnPush, signals para estado, AsyncPipe para observables.

---

## 8. Forms

- Nuevas apps con signals: Signal Forms (experimental, v21+).
- Producción: Reactive Forms con `FormBuilder`, `fb.nonNullable.group()`, `getRawValue()`.

---

## 9. Performance

- **Imágenes**: `NgOptimizedImage`, `ngSrc`, siempre `width`/`height` (o `fill`); `priority` para LCP.
- **Lazy**: `@defer` (viewport, interaction, idle, timer, when).
- **Rutas**: lazy con `loadComponent` / `loadChildren`.
- **SSR**: `provideClientHydration()` cuando el documento lo indique (SEO, etc.).

---

## 10. Comandos

```bash
ng new my-app --style=scss --ssr=false
ng g c features/products/components/product-card --flat
ng g s features/products/services/product --flat
ng g g core/guards/auth --functional
```

---

## Recursos

- https://angular.dev/style-guide
- https://angular.dev/guide/signals
- https://angular.dev/guide/templates/control-flow
- https://angular.dev/guide/zoneless
- https://angular.dev/guide/image-optimization
- https://angular.dev/guide/defer
