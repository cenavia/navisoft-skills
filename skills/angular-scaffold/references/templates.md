# Plantillas de Archivos Angular

## Tabla de Contenidos

1. [package.json](#packagejson)
2. [angular.json](#angularjson)
3. [tsconfig.json](#tsconfigjson)
4. [.eslintrc.json](#eslintrcjson)
5. [tailwind.config.js](#tailwindconfigjs)
6. [App Module](#app-module)
7. [Shared Module](#shared-module)
8. [Feature Module Completo](#feature-module-completo)

---

## package.json

```json
{
  "name": "{{PROJECT_NAME}}",
  "version": "0.0.0",
  "scripts": {
    "ng": "ng",
    "start": "ng serve",
    "build": "ng build",
    "watch": "ng build --watch --configuration development",
    "test": "ng test",
    "lint": "ng lint"
  },
  "private": true,
  "dependencies": {
    "@angular/animations": "~14.1.0",
    "@angular/common": "~14.1.0",
    "@angular/compiler": "~14.1.0",
    "@angular/core": "~14.1.0",
    "@angular/forms": "~14.1.0",
    "@angular/platform-browser": "~14.1.0",
    "@angular/platform-browser-dynamic": "~14.1.0",
    "@angular/router": "~14.1.0",
    "rxjs": "~7.5.0",
    "tslib": "^2.3.0",
    "zone.js": "~0.11.4"
  },
  "devDependencies": {
    "@angular-devkit/build-angular": "~14.1.0",
    "@angular-eslint/builder": "14.1.2",
    "@angular-eslint/eslint-plugin": "14.1.2",
    "@angular-eslint/eslint-plugin-template": "14.1.2",
    "@angular-eslint/schematics": "14.1.2",
    "@angular-eslint/template-parser": "14.1.2",
    "@angular/cli": "~14.1.0",
    "@angular/compiler-cli": "~14.1.0",
    "@types/jasmine": "~4.0.0",
    "@typescript-eslint/eslint-plugin": "5.29.0",
    "@typescript-eslint/parser": "5.29.0",
    "eslint": "^8.18.0",
    "jasmine-core": "~4.2.0",
    "karma": "~6.4.0",
    "karma-chrome-launcher": "~3.1.0",
    "karma-coverage": "~2.2.0",
    "karma-jasmine": "~5.1.0",
    "karma-jasmine-html-reporter": "~2.0.0",
    "typescript": "~4.7.2"
  }
}
```

### Con TailwindCSS

Agregar a dependencies:
```json
{
  "devDependencies": {
    "autoprefixer": "^10.4.7",
    "postcss": "^8.4.14",
    "tailwindcss": "^3.1.0",
    "@tailwindcss/forms": "^0.5.2"
  }
}
```

### Con JWT y Cookies

Agregar a dependencies:
```json
{
  "dependencies": {
    "jwt-decode": "^3.1.2",
    "typescript-cookie": "^1.0.0"
  }
}
```

### Con FontAwesome

Agregar a dependencies:
```json
{
  "dependencies": {
    "@fortawesome/angular-fontawesome": "^0.11.0",
    "@fortawesome/fontawesome-svg-core": "^6.1.0",
    "@fortawesome/free-brands-svg-icons": "^6.1.0",
    "@fortawesome/free-regular-svg-icons": "^6.1.0",
    "@fortawesome/free-solid-svg-icons": "^6.1.0"
  }
}
```

---

## angular.json

```json
{
  "$schema": "./node_modules/@angular/cli/lib/config/schema.json",
  "version": 1,
  "newProjectRoot": "projects",
  "projects": {
    "{{PROJECT_NAME}}": {
      "projectType": "application",
      "schematics": {
        "@schematics/angular:component": {
          "style": "scss"
        }
      },
      "root": "",
      "sourceRoot": "src",
      "prefix": "app",
      "architect": {
        "build": {
          "builder": "@angular-devkit/build-angular:browser",
          "options": {
            "outputPath": "dist/{{PROJECT_NAME}}",
            "index": "src/index.html",
            "main": "src/main.ts",
            "polyfills": "src/polyfills.ts",
            "tsConfig": "tsconfig.app.json",
            "inlineStyleLanguage": "scss",
            "assets": [
              "src/favicon.ico",
              "src/assets"
            ],
            "styles": [
              "src/styles.scss"
            ],
            "scripts": []
          },
          "configurations": {
            "production": {
              "budgets": [
                {
                  "type": "initial",
                  "maximumWarning": "500kb",
                  "maximumError": "1mb"
                },
                {
                  "type": "anyComponentStyle",
                  "maximumWarning": "2kb",
                  "maximumError": "4kb"
                }
              ],
              "fileReplacements": [
                {
                  "replace": "src/environments/environment.ts",
                  "with": "src/environments/environment.prod.ts"
                }
              ],
              "outputHashing": "all"
            },
            "development": {
              "buildOptimizer": false,
              "optimization": false,
              "vendorChunk": true,
              "extractLicenses": false,
              "sourceMap": true,
              "namedChunks": true
            }
          },
          "defaultConfiguration": "production"
        },
        "serve": {
          "builder": "@angular-devkit/build-angular:dev-server",
          "configurations": {
            "production": {
              "browserTarget": "{{PROJECT_NAME}}:build:production"
            },
            "development": {
              "browserTarget": "{{PROJECT_NAME}}:build:development"
            }
          },
          "defaultConfiguration": "development"
        },
        "test": {
          "builder": "@angular-devkit/build-angular:karma",
          "options": {
            "main": "src/test.ts",
            "polyfills": "src/polyfills.ts",
            "tsConfig": "tsconfig.spec.json",
            "karmaConfig": "karma.conf.js",
            "inlineStyleLanguage": "scss",
            "assets": [
              "src/favicon.ico",
              "src/assets"
            ],
            "styles": [
              "src/styles.scss"
            ],
            "scripts": []
          }
        },
        "lint": {
          "builder": "@angular-eslint/builder:lint",
          "options": {
            "lintFilePatterns": [
              "src/**/*.ts",
              "src/**/*.html"
            ]
          }
        }
      }
    }
  },
  "cli": {
    "analytics": false
  }
}
```

---

## tsconfig.json

```json
{
  "compileOnSave": false,
  "compilerOptions": {
    "baseUrl": "./",
    "outDir": "./dist/out-tsc",
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "noImplicitOverride": true,
    "noPropertyAccessFromIndexSignature": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "sourceMap": true,
    "declaration": false,
    "downlevelIteration": true,
    "experimentalDecorators": true,
    "moduleResolution": "node",
    "importHelpers": true,
    "target": "ES2020",
    "module": "ES2020",
    "lib": ["ES2020", "dom"],
    "paths": {
      "@services/*": ["src/app/services/*"],
      "@models/*": ["src/app/models/*"],
      "@guards/*": ["src/app/guards/*"],
      "@interceptors/*": ["src/app/interceptors/*"],
      "@utils/*": ["src/app/utils/*"],
      "@shared/*": ["src/app/shared/*"],
      "@environments/*": ["src/environments/*"]
    }
  },
  "angularCompilerOptions": {
    "enableI18nLegacyMessageIdFormat": false,
    "strictInjectionParameters": true,
    "strictInputAccessModifiers": true,
    "strictTemplates": true
  }
}
```

---

## .eslintrc.json

```json
{
  "root": true,
  "ignorePatterns": ["projects/**/*"],
  "overrides": [
    {
      "files": ["*.ts"],
      "parserOptions": {
        "project": ["tsconfig.json"],
        "createDefaultProgram": true
      },
      "extends": [
        "plugin:@angular-eslint/recommended",
        "plugin:@angular-eslint/template/process-inline-templates"
      ],
      "rules": {
        "@angular-eslint/directive-selector": [
          "error",
          {
            "type": "attribute",
            "prefix": "app",
            "style": "camelCase"
          }
        ],
        "@angular-eslint/component-selector": [
          "error",
          {
            "type": "element",
            "prefix": "app",
            "style": "kebab-case"
          }
        ]
      }
    },
    {
      "files": ["*.html"],
      "extends": ["plugin:@angular-eslint/template/recommended"],
      "rules": {}
    }
  ]
}
```

---

## tailwind.config.js

```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./src/**/*.{html,ts}",
  ],
  theme: {
    extend: {},
  },
  plugins: [
    require('@tailwindcss/forms'),
  ],
}
```

### styles.scss con Tailwind

```scss
@tailwind base;
@tailwind components;
@tailwind utilities;

// Custom styles below
```

---

## App Module

```typescript
// src/app/app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { HttpClientModule, HTTP_INTERCEPTORS } from '@angular/common/http';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { TokenInterceptor } from '@interceptors/token.interceptor';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    HttpClientModule,
    AppRoutingModule
  ],
  providers: [
    {
      provide: HTTP_INTERCEPTORS,
      useClass: TokenInterceptor,
      multi: true
    }
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

### App Component

```typescript
// src/app/app.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss']
})
export class AppComponent {
  title = '{{PROJECT_NAME}}';
}
```

```html
<!-- src/app/app.component.html -->
<router-outlet></router-outlet>
```

### App Routing Module

```typescript
// src/app/app-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { AuthGuard } from '@guards/auth.guard';

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
  {
    path: '',
    redirectTo: '/auth/login',
    pathMatch: 'full'
  },
  {
    path: '**',
    redirectTo: '/auth/login'
  }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

---

## Shared Module

```typescript
// src/app/shared/shared.module.ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ReactiveFormsModule, FormsModule } from '@angular/forms';
import { RouterModule } from '@angular/router';

// Components
// import { ButtonComponent } from './components/button/button.component';

// Directives
// import { HighlightDirective } from './directives/highlight.directive';

// Pipes
// import { FormatDatePipe } from './pipes/format-date.pipe';

const COMPONENTS = [
  // ButtonComponent,
];

const DIRECTIVES = [
  // HighlightDirective,
];

const PIPES = [
  // FormatDatePipe,
];

@NgModule({
  declarations: [
    ...COMPONENTS,
    ...DIRECTIVES,
    ...PIPES,
  ],
  imports: [
    CommonModule,
    ReactiveFormsModule,
    FormsModule,
    RouterModule,
  ],
  exports: [
    // Re-export common modules
    CommonModule,
    ReactiveFormsModule,
    FormsModule,
    RouterModule,
    // Export declarations
    ...COMPONENTS,
    ...DIRECTIVES,
    ...PIPES,
  ]
})
export class SharedModule { }
```

---

## Feature Module Completo

### Auth Module (ejemplo)

```typescript
// src/app/modules/auth/auth.module.ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ReactiveFormsModule } from '@angular/forms';

import { AuthRoutingModule } from './auth-routing.module';
import { LoginComponent } from './pages/login/login.component';
import { RegisterComponent } from './pages/register/register.component';
import { ForgotPasswordComponent } from './pages/forgot-password/forgot-password.component';

@NgModule({
  declarations: [
    LoginComponent,
    RegisterComponent,
    ForgotPasswordComponent,
  ],
  imports: [
    CommonModule,
    ReactiveFormsModule,
    AuthRoutingModule
  ]
})
export class AuthModule { }
```

```typescript
// src/app/modules/auth/auth-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

import { LoginComponent } from './pages/login/login.component';
import { RegisterComponent } from './pages/register/register.component';
import { ForgotPasswordComponent } from './pages/forgot-password/forgot-password.component';

const routes: Routes = [
  {
    path: 'login',
    component: LoginComponent
  },
  {
    path: 'register',
    component: RegisterComponent
  },
  {
    path: 'forgot-password',
    component: ForgotPasswordComponent
  },
  {
    path: '',
    redirectTo: 'login',
    pathMatch: 'full'
  }
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class AuthRoutingModule { }
```

### Page Component Template

```typescript
// src/app/modules/auth/pages/login/login.component.ts
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';
import { Router } from '@angular/router';
import { AuthService } from '@services/auth.service';

@Component({
  selector: 'app-login',
  templateUrl: './login.component.html',
  styleUrls: ['./login.component.scss']
})
export class LoginComponent implements OnInit {
  form!: FormGroup;
  isLoading = false;
  errorMessage: string | null = null;

  constructor(
    private fb: FormBuilder,
    private authService: AuthService,
    private router: Router
  ) {}

  ngOnInit(): void {
    this.buildForm();
  }

  private buildForm(): void {
    this.form = this.fb.group({
      email: ['', [Validators.required, Validators.email]],
      password: ['', [Validators.required, Validators.minLength(6)]]
    });
  }

  onSubmit(): void {
    if (this.form.invalid) {
      this.form.markAllAsTouched();
      return;
    }
    
    this.isLoading = true;
    this.errorMessage = null;
    
    // TODO: Implement login logic
    const { email, password } = this.form.value;
    this.authService.login(email, password).subscribe({
      next: () => {
        this.router.navigate(['/app']);
      },
      error: (err) => {
        this.errorMessage = err.message || 'Error al iniciar sesión';
        this.isLoading = false;
      }
    });
  }
}
```

```html
<!-- src/app/modules/auth/pages/login/login.component.html -->
<div class="min-h-screen flex items-center justify-center bg-gray-100">
  <div class="max-w-md w-full bg-white rounded-lg shadow-md p-8">
    <h2 class="text-2xl font-bold text-center text-gray-800 mb-6">
      Iniciar Sesión
    </h2>
    
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <!-- Email -->
      <div class="mb-4">
        <label for="email" class="block text-sm font-medium text-gray-700 mb-1">
          Email
        </label>
        <input
          type="email"
          id="email"
          formControlName="email"
          class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          [class.border-red-500]="form.get('email')?.invalid && form.get('email')?.touched"
        />
        <p *ngIf="form.get('email')?.hasError('required') && form.get('email')?.touched"
           class="text-red-500 text-xs mt-1">
          El email es requerido
        </p>
        <p *ngIf="form.get('email')?.hasError('email') && form.get('email')?.touched"
           class="text-red-500 text-xs mt-1">
          Ingresa un email válido
        </p>
      </div>

      <!-- Password -->
      <div class="mb-6">
        <label for="password" class="block text-sm font-medium text-gray-700 mb-1">
          Contraseña
        </label>
        <input
          type="password"
          id="password"
          formControlName="password"
          class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          [class.border-red-500]="form.get('password')?.invalid && form.get('password')?.touched"
        />
        <p *ngIf="form.get('password')?.hasError('required') && form.get('password')?.touched"
           class="text-red-500 text-xs mt-1">
          La contraseña es requerida
        </p>
      </div>

      <!-- Error message -->
      <div *ngIf="errorMessage" class="mb-4 p-3 bg-red-100 text-red-700 rounded-md text-sm">
        {{ errorMessage }}
      </div>

      <!-- Submit -->
      <button
        type="submit"
        [disabled]="isLoading"
        class="w-full py-2 px-4 bg-blue-600 text-white font-medium rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 disabled:opacity-50 disabled:cursor-not-allowed"
      >
        <span *ngIf="!isLoading">Iniciar Sesión</span>
        <span *ngIf="isLoading">Cargando...</span>
      </button>
    </form>

    <!-- Links -->
    <div class="mt-4 text-center text-sm">
      <a routerLink="/auth/forgot-password" class="text-blue-600 hover:underline">
        ¿Olvidaste tu contraseña?
      </a>
    </div>
    <div class="mt-2 text-center text-sm">
      <span class="text-gray-600">¿No tienes cuenta?</span>
      <a routerLink="/auth/register" class="text-blue-600 hover:underline ml-1">
        Regístrate
      </a>
    </div>
  </div>
</div>
```
