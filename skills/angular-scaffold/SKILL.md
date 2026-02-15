---
name: angular-scaffold
description: |
  Genera esqueletos de proyectos Angular de forma precisa y exhaustiva siguiendo documentos de arquitectura.
  Usar cuando el usuario solicite:
  (1) Crear/generar un proyecto Angular desde cero basado en especificaciones de arquitectura
  (2) Scaffolding de módulos, servicios, guards, interceptores según un documento técnico
  (3) Estructura inicial de proyecto Angular con configuración de rutas, aliases y módulos lazy-loaded
  (4) Generar archivos base (models, services, components) respetando convenciones de un proyecto existente
  Soporta Angular 14+ con TypeScript estricto, TailwindCSS, y patrones enterprise.
---

# Angular Scaffold Generator

Skill para generar esqueletos de proyectos Angular siguiendo documentos de arquitectura.

## Tabla de Contenidos

1. [Flujo de Trabajo Principal](#flujo-de-trabajo-principal)
2. [Análisis del Documento de Arquitectura](#análisis-del-documento-de-arquitectura)
3. [Generación de Estructura](#generación-de-estructura)
4. [Patrones de Código](#patrones-de-código)
5. [Validación Final](#validación-final)

---

## Flujo de Trabajo Principal

```
1. ANALIZAR documento de arquitectura → extraer requisitos
2. PLANIFICAR estructura de carpetas y archivos
3. GENERAR configuración base (angular.json, tsconfig, package.json)
4. CREAR módulos feature con lazy loading
5. IMPLEMENTAR servicios, guards, interceptores
6. GENERAR modelos e interfaces TypeScript
7. CONFIGURAR rutas y aliases
8. VALIDAR coherencia del scaffold
```

---

## Análisis del Documento de Arquitectura

### Extraer del Documento

| Categoría | Qué buscar | Impacto en scaffold |
|-----------|------------|---------------------|
| **Tech Stack** | Versión Angular, libs adicionales | `package.json`, `angular.json` |
| **Convenciones** | Prefijos selectores, aliases imports | `tsconfig.json`, `.eslintrc` |
| **Módulos** | Features, lazy loading, shared | Estructura `src/app/modules/` |
| **Servicios** | APIs, estado, autenticación | `src/app/services/` |
| **Seguridad** | Guards, interceptores, tokens | `src/app/guards/`, `src/app/interceptors/` |
| **Modelos** | Entidades de dominio | `src/app/models/` |
| **Estilos** | CSS framework, preprocesador | `styles.scss`, `tailwind.config.js` |

### Checklist de Requisitos

```markdown
[ ] Versión Angular especificada
[ ] TypeScript strict mode definido
[ ] Módulos feature identificados
[ ] Estrategia de routing (lazy/eager)
[ ] Servicios core listados
[ ] Guards necesarios
[ ] Interceptores HTTP requeridos
[ ] Modelos de dominio definidos
[ ] Aliases de imports configurados
[ ] Framework CSS elegido
```

---

## Generación de Estructura

### Estructura Base Estándar

```
src/
├── app/
│   ├── guards/                    # Route guards
│   │   └── auth.guard.ts
│   ├── interceptors/              # HTTP interceptors
│   │   └── token.interceptor.ts
│   ├── models/                    # TypeScript interfaces/types
│   │   └── index.ts
│   ├── services/                  # Core services
│   │   └── index.ts
│   ├── modules/                   # Feature modules (lazy-loaded)
│   │   ├── auth/
│   │   │   ├── auth.module.ts
│   │   │   ├── auth-routing.module.ts
│   │   │   ├── pages/
│   │   │   └── components/
│   │   └── layout/
│   │       ├── layout.module.ts
│   │       ├── layout-routing.module.ts
│   │       ├── pages/
│   │       └── components/
│   ├── shared/                    # Shared module
│   │   ├── shared.module.ts
│   │   ├── components/
│   │   ├── directives/
│   │   └── pipes/
│   ├── utils/                     # Utility functions
│   ├── app.component.ts
│   ├── app.module.ts
│   └── app-routing.module.ts
├── environments/
│   ├── environment.ts
│   └── environment.prod.ts
├── assets/
└── styles.scss
```

### Reglas de Nomenclatura

| Tipo | Patrón | Ejemplo |
|------|--------|---------|
| Componente | `kebab-case.component.ts` | `user-profile.component.ts` |
| Servicio | `kebab-case.service.ts` | `auth.service.ts` |
| Guard | `kebab-case.guard.ts` | `auth.guard.ts` |
| Interceptor | `kebab-case.interceptor.ts` | `token.interceptor.ts` |
| Modelo | `kebab-case.model.ts` | `user.model.ts` |
| Módulo | `kebab-case.module.ts` | `auth.module.ts` |
| Directiva | `kebab-case.directive.ts` | `highlight.directive.ts` |
| Pipe | `kebab-case.pipe.ts` | `format-date.pipe.ts` |

### Selectores Angular

```typescript
// Componentes: elemento con prefijo 'app-'
@Component({
  selector: 'app-user-profile',  // kebab-case
  ...
})

// Directivas: atributo con prefijo 'app'
@Directive({
  selector: '[appHighlight]',    // camelCase
  ...
})
```

---

## Patrones de Código

### tsconfig.json - Path Aliases

```json
{
  "compilerOptions": {
    "strict": true,
    "baseUrl": "./",
    "paths": {
      "@services/*": ["src/app/services/*"],
      "@models/*": ["src/app/models/*"],
      "@guards/*": ["src/app/guards/*"],
      "@interceptors/*": ["src/app/interceptors/*"],
      "@utils/*": ["src/app/utils/*"],
      "@shared/*": ["src/app/shared/*"],
      "@environments/*": ["src/environments/*"],
      "@auth/*": ["src/app/modules/auth/*"],
      "@layout/*": ["src/app/modules/layout/*"]
    }
  }
}
```

### Feature Module con Lazy Loading

```typescript
// feature.module.ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FeatureRoutingModule } from './feature-routing.module';

@NgModule({
  declarations: [],
  imports: [
    CommonModule,
    FeatureRoutingModule
  ]
})
export class FeatureModule { }
```

```typescript
// feature-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class FeatureRoutingModule { }
```

### App Routing con Lazy Loading

```typescript
// app-routing.module.ts
const routes: Routes = [
  {
    path: 'auth',
    loadChildren: () => import('./modules/auth/auth.module')
      .then(m => m.AuthModule)
  },
  {
    path: 'app',
    canActivate: [AuthGuard],
    loadChildren: () => import('./modules/layout/layout.module')
      .then(m => m.LayoutModule)
  },
  { path: '', redirectTo: '/auth/login', pathMatch: 'full' },
  { path: '**', redirectTo: '/auth/login' }
];
```

### Service Pattern

```typescript
// ejemplo.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable, BehaviorSubject } from 'rxjs';
import { environment } from '@environments/environment';

@Injectable({
  providedIn: 'root'
})
export class EjemploService {
  private apiUrl = `${environment.API_URL}/api/v1`;
  
  // Estado reactivo cuando se necesita
  private _data$ = new BehaviorSubject<Data | null>(null);
  readonly data$ = this._data$.asObservable();

  constructor(private http: HttpClient) {}

  getAll(): Observable<Data[]> {
    return this.http.get<Data[]>(`${this.apiUrl}/recurso`);
  }
}
```

### Auth Guard Pattern

```typescript
// auth.guard.ts
import { Injectable } from '@angular/core';
import { CanActivate, Router, UrlTree } from '@angular/router';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';
import { AuthService } from '@services/auth.service';

@Injectable({
  providedIn: 'root'
})
export class AuthGuard implements CanActivate {
  constructor(
    private authService: AuthService,
    private router: Router
  ) {}

  canActivate(): Observable<boolean | UrlTree> {
    return this.authService.user$.pipe(
      map(user => user ? true : this.router.createUrlTree(['/auth/login']))
    );
  }
}
```

### HTTP Interceptor con Token

```typescript
// token.interceptor.ts
import { Injectable } from '@angular/core';
import {
  HttpInterceptor, HttpRequest, HttpHandler,
  HttpEvent, HttpContext, HttpContextToken
} from '@angular/common/http';
import { Observable, switchMap } from 'rxjs';
import { AuthService } from '@services/auth.service';

const CHECK_TOKEN = new HttpContextToken<boolean>(() => false);

export function checkToken() {
  return new HttpContext().set(CHECK_TOKEN, true);
}

@Injectable()
export class TokenInterceptor implements HttpInterceptor {
  constructor(private authService: AuthService) {}

  intercept(req: HttpRequest<unknown>, next: HttpHandler): Observable<HttpEvent<unknown>> {
    if (req.context.get(CHECK_TOKEN)) {
      return this.addToken(req, next);
    }
    return next.handle(req);
  }

  private addToken(req: HttpRequest<unknown>, next: HttpHandler): Observable<HttpEvent<unknown>> {
    const token = this.authService.getAccessToken();
    if (token) {
      const authReq = req.clone({
        headers: req.headers.set('Authorization', `Bearer ${token}`)
      });
      return next.handle(authReq);
    }
    return next.handle(req);
  }
}
```

### Model Interface Pattern

```typescript
// user.model.ts
export interface User {
  id: string;
  email: string;
  name: string;
  createdAt: Date;
  updatedAt: Date;
}

export interface CreateUserDto {
  email: string;
  password: string;
  name: string;
}

export interface UpdateUserDto {
  name?: string;
  email?: string;
}
```

### Environment Configuration

```typescript
// environment.ts
export const environment = {
  production: false,
  API_URL: 'http://localhost:3000'
};

// environment.prod.ts
export const environment = {
  production: true,
  API_URL: 'https://api.production.com'
};
```

---

## Validación Final

### Checklist de Scaffold Completo

```markdown
## Configuración
[ ] angular.json configurado correctamente
[ ] tsconfig.json con strict: true y aliases
[ ] package.json con dependencias correctas
[ ] .eslintrc.json con reglas Angular

## Estructura
[ ] Carpetas según arquitectura definida
[ ] Módulos feature creados
[ ] Shared module configurado
[ ] Services en ubicación correcta

## Código
[ ] Selectores siguen convención (app-* / app*)
[ ] Imports usan aliases (@services/*, etc.)
[ ] Services retornan Observable<T>
[ ] Guards implementan CanActivate
[ ] Interceptors registrados en providers

## Routing
[ ] Lazy loading configurado
[ ] Guards aplicados a rutas protegidas
[ ] Redirects por defecto definidos

## Tipos
[ ] Modelos TypeScript definidos
[ ] Sin uso de 'any'
[ ] Interfaces exportadas
```

### Comandos de Verificación

```bash
# Verificar que compila
ng build --configuration=development

# Verificar linting
ng lint

# Verificar tests (si aplica)
ng test --no-watch --browsers=ChromeHeadless
```

---

## Instrucciones de Ejecución

1. **Leer completamente** el documento de arquitectura provisto
2. **Identificar** todos los elementos a generar (módulos, servicios, modelos, etc.)
3. **Crear estructura** de carpetas según el patrón definido
4. **Generar archivos** de configuración (tsconfig, angular.json, etc.)
5. **Implementar** módulos con sus rutas y componentes placeholder
6. **Crear** servicios con métodos stub que retornen Observable
7. **Definir** guards e interceptores según requisitos de seguridad
8. **Generar** modelos/interfaces TypeScript para todas las entidades
9. **Configurar** estilos base (Tailwind si aplica)
10. **Validar** que la estructura compila sin errores

### Notas Importantes

- **NO generar lógica de negocio completa** - solo stubs y estructura
- **SÍ generar imports** correctos entre archivos
- **SÍ incluir** comentarios TODO donde se requiera implementación
- **Respetar** versión de Angular especificada en el documento
- **Mantener** consistencia con convenciones del proyecto existente
