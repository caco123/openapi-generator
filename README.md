# OpenapiGenerator

This project was generated using [Angular CLI](https://github.com/angular/angular-cli) version 21.2.0.

## Development server

To start a local development server, run:

```bash
ng serve
```

Once the server is running, open your browser and navigate to `http://localhost:4200/`. The application will automatically reload whenever you modify any of the source files.

## Code scaffolding

Angular CLI includes powerful code scaffolding tools. To generate a new component, run:

```bash
ng generate component component-name
```

For a complete list of available schematics (such as `components`, `directives`, or `pipes`), run:

```bash
ng generate --help
```

## Building

To build the project run:

```bash
ng build
```

This will compile your project and store the build artifacts in the `dist/` directory. By default, the production build optimizes your application for performance and speed.

## Running unit tests

To execute unit tests with the [Vitest](https://vitest.dev/) test runner, use the following command:

```bash
ng test
```

## Running end-to-end tests

For end-to-end (e2e) testing, run:

```bash
ng e2e
```

Angular CLI does not come with an end-to-end testing framework by default. You can choose one that suits your needs.

## Additional Resources

For more information on using the Angular CLI, including detailed command references, visit the [Angular CLI Overview and Command Reference](https://angular.dev/tools/cli) page.

---

## Uso en una aplicación Angular

A continuación se detalla cómo configurar y utilizar el cliente generado (`dummy-openapi`) en una aplicación Angular basada en componentes *standalone* y `app.config.ts`.

### 1. Instalación

Instala el paquete en tu proyecto:

```bash
npm install dummy-openapi
```

### 2. Configuración en `app.config.ts`

Para configurar la URL base de los endpoints y los parámetros globales del cliente API, define un proveedor de tipo `FactoryProvider` para la clase `Configuration` de `dummy-openapi` y agrégalo al arreglo `providers` de `appConfig`:

```typescript
import { ApplicationConfig, FactoryProvider, provideBrowserGlobalErrorListeners } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';
import { provideClientHydration } from '@angular/platform-browser';
import { Configuration } from 'dummy-openapi';

import { routes } from './app.routes';

// Proveedor de configuración del cliente API
const ApiConfigurationProvider: FactoryProvider = {
  provide: Configuration,
  useFactory: () => {
    return new Configuration({
      basePath: 'https://api.example.com', // Reemplaza por la URL de tu API
    });
  },
};

export const appConfig: ApplicationConfig = {
  providers: [
    provideBrowserGlobalErrorListeners(),
    provideRouter(routes),
    provideClientHydration(),
    provideHttpClient(), // Requerido para la comunicación HTTP del cliente
    ApiConfigurationProvider,
  ],
};
```

> **Nota:** La librería hace uso de `HttpClient` internamente, por lo que es imprescindible incluir `provideHttpClient()` (de `@angular/common/http`) en los proveedores de la aplicación si no está presente.

### 3. Consumo en un Componente o Servicio

Una vez configurado `ApiConfigurationProvider`, los servicios del API (como `UsersService`) inyectarán automáticamente esta configuración con el `basePath` indicado:

```typescript
import { Component, inject, OnInit } from '@angular/core';
import { UserListResponse, UsersService } from 'dummy-openapi';

@Component({
  selector: 'app-users',
  standalone: true,
  template: `
    @if (users()) {
      <ul>
        @for (user of users().items; track user.id) {
          <li>{{ user.name }} ({{ user.email }})</li>
        }
      </ul>
    }
  `,
})
export class UsersComponent implements OnInit {
  private readonly usersService = inject(UsersService);  
  private readonly users = toSignal<UserListResponse>(this.usersService.usersGet());
}
```
