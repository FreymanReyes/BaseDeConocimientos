# 🔐 Autenticación JWT, Guards, Interceptores y Formularios Reactivos con Signals

> **Proyecto:** `pokedex-ng` — Angular 21
> **Prerequisito:** Tener completadas las Fases 1, 2 y 3 (entorno, arquitectura y dependencias configuradas)

---

## 🎯 ¿Qué construimos en esta fase?

Un sistema de autenticación completo que incluye:

| Pieza | Archivo | Responsabilidad |
|---|---|---|
| Model | `core/models/auth.model.ts` | Define la forma de los datos |
| Service | `core/services/auth.service.ts` | Lógica de autenticación y estado |
| Auth Guard | `core/guards/auth.guard.ts` | Protege rutas privadas |
| Public Guard | `core/guards/public.guard.ts` | Protege rutas públicas |
| Interceptor | `core/interceptors/auth.interceptor.ts` | Adjunta el token a las peticiones |
| Login Page | `features/auth/pages/login/login.ts` | Formulario con validaciones |

---

## 🧠 ¿Cómo funciona la autenticación con JWT?

```
1. Usuario llena el formulario → username + password
2. Angular envía esos datos a FakeStore API
3. FakeStore API responde con un TOKEN JWT
4. Angular guarda ese token en localStorage y en un Signal
5. En cada petición a FakeStore el interceptor adjunta el token
6. Si el token no existe → el Guard bloquea el acceso a rutas protegidas
```

> 💡 **¿Qué es un JWT (JSON Web Token)?** Es una cadena de texto cifrada que el servidor entrega cuando te autenticas correctamente. Es como un brazalete de entrada a un evento — mientras lo tengas, puedes entrar a las áreas autorizadas.
>
> Un JWT tiene tres partes separadas por puntos:
> ```
> eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9   ← Header (algoritmo de cifrado)
> .eyJzdWIiOiIxMjM0NTY3ODkwIn0             ← Payload (datos del usuario)
> .dozjgNryP4J3jVmNHl0w5N_XgL0n3I9PlFUP0  ← Signature (firma de validez)
> ```

---

## 📡 La API utilizada — FakeStore API

```
POST https://fakestoreapi.com/auth/login

Body:
{
  "username": "mor_2314",
  "password": "83r5^_"
}

Respuesta exitosa:
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

> 💡 Estos son usuarios reales de FakeStore API. No requiere registro — existen en su base de datos. El usuario `mor_2314` con esa contraseña siempre funciona para pruebas.

---

## 📁 Paso 1 — El Modelo de Autenticación

**Archivo:** `src/app/core/models/auth.model.ts`

```typescript
export interface LoginRequest {
  username: string;
  password: string;
}

export interface LoginResponse {
  token: string;
}

export interface AuthUser {
  username: string;
  token: string;
}
```

> 💡 **¿Por qué tres interfaces separadas?**
>
> - `LoginRequest` → lo que **enviamos** a la API (username + password)
> - `LoginResponse` → lo que la API **nos devuelve** (solo el token)
> - `AuthUser` → lo que **guardamos** en nuestra app (username + token juntos)
>
> Separar estas tres cosas sigue el principio de **responsabilidad única** — cada interfaz describe un momento distinto del flujo. Si mañana la API cambia y devuelve más datos, solo tocas `LoginResponse` sin afectar el resto.

---

## 📁 Paso 2 — El Auth Service

**Archivo:** `src/app/core/services/auth.service.ts`

```typescript
import { inject, Injectable, signal, computed } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Router } from '@angular/router';
import { tap } from 'rxjs/operators';
import { LoginRequest, LoginResponse, AuthUser } from '../models/auth.model';

@Injectable({
  providedIn: 'root'
})
export class AuthService {

  private readonly http = inject(HttpClient);
  private readonly router = inject(Router);

  private readonly API_URL = 'https://fakestoreapi.com';
  private readonly TOKEN_KEY = 'auth_token';
  private readonly USERNAME_KEY = 'auth_username';

  // --- Signals ---
  private readonly _currentUser = signal<AuthUser | null>(
    this.loadUserFromStorage()
  );

  readonly currentUser = this._currentUser.asReadonly();
  readonly isAuthenticated = computed(() => this._currentUser() !== null);

  // --- Métodos ---
  login(credentials: LoginRequest) {
    return this.http.post<LoginResponse>(
      `${this.API_URL}/auth/login`,
      credentials
    ).pipe(
      tap(response => {
        const user: AuthUser = {
          username: credentials.username,
          token: response.token
        };
        this.saveUserToStorage(user);
        this._currentUser.set(user);
      })
    );
  }

  logout(): void {
    localStorage.removeItem(this.TOKEN_KEY);
    localStorage.removeItem(this.USERNAME_KEY);
    this._currentUser.set(null);
    this.router.navigate(['/login']);
  }

  private saveUserToStorage(user: AuthUser): void {
    localStorage.setItem(this.TOKEN_KEY, user.token);
    localStorage.setItem(this.USERNAME_KEY, user.username);
  }

  private loadUserFromStorage(): AuthUser | null {
    const token = localStorage.getItem(this.TOKEN_KEY);
    const username = localStorage.getItem(this.USERNAME_KEY);
    if (token && username) {
      return { token, username };
    }
    return null;
  }
}
```

### Explicación de las piezas clave

#### `@Injectable({ providedIn: 'root' })`
> 💡 Le dice a Angular que este servicio existe **una sola vez en toda la app** (Singleton). Se crea cuando la app arranca y se destruye cuando se cierra. Todos los componentes que lo inyecten reciben la misma instancia.

#### `inject()` — inyección de dependencias moderna
> 💡 En Angular 21 se usa `inject()` en lugar del constructor tradicional. En lugar de crear `new HttpClient()` manualmente, Angular entrega la instancia que ya existe. Más limpio y moderno que el enfoque de constructor.

#### Los tres Signals

```typescript
// 1️⃣ PRIVADO — solo el servicio puede modificarlo
private readonly _currentUser = signal<AuthUser | null>(
  this.loadUserFromStorage()
);

// 2️⃣ SOLO LECTURA — los componentes pueden leerlo pero no modificarlo
readonly currentUser = this._currentUser.asReadonly();

// 3️⃣ COMPUTED — se calcula automáticamente cuando _currentUser cambia
readonly isAuthenticated = computed(() => this._currentUser() !== null);
```

> 💡 **¿Por qué esta separación?** Es el principio de **encapsulamiento**:
> - El estado solo se modifica desde adentro del servicio
> - Los componentes pueden leer pero no escribir
> - `isAuthenticated` se actualiza solo — nunca puede quedar desincronizado

#### El método `login()` y RxJS

> 💡 **¿Por qué retorna el Observable en lugar de subscribirse aquí?**
> El servicio define **qué hacer** pero no decide **cuándo hacerlo**. El componente decide cuándo ejecutar la petición y maneja los estados de carga y error. Si el servicio se subscribiera solo, el componente nunca sabría el resultado.
>
> **`tap()`** ejecuta efectos secundarios (guardar token, actualizar Signal) sin modificar el valor que fluye por el Observable. Es como espiar el resultado sin interrumpirlo.

#### `localStorage` — persistencia del token

> 💡 Los Signals son **memoria RAM** — rápidos pero volátiles, se pierden al recargar la página. `localStorage` es **disco duro** — persiste aunque recargues o cierres la pestaña.
>
> Al arrancar la app, `loadUserFromStorage()` recupera el token guardado y rehidrata el Signal automáticamente. Así el usuario no pierde su sesión al recargar.
>
> Se usan constantes (`TOKEN_KEY`, `USERNAME_KEY`) en lugar de strings directos para evitar errores de typo — TypeScript avisa si escribes mal el nombre de una constante, pero no avisa si escribes mal un string.

---

## 📁 Paso 3 — Auth Guard (protege rutas privadas)

**Archivo:** `src/app/core/guards/auth.guard.ts`

```typescript
import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';
import { AuthService } from '../services/auth.service';

export const authGuard: CanActivateFn = () => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.isAuthenticated()) {
    return true;
  }

  return router.createUrlTree(['/login']);
};
```

> 💡 **¿Qué es un Guard?** Es una función que Angular ejecuta **antes de activar una ruta**. Decide si el usuario puede entrar o no.
>
> - `return true` → deja pasar
> - `return router.createUrlTree(['/login'])` → redirige a `/login`
>
> Se usa `createUrlTree()` en lugar de `return false` porque así Angular sabe a dónde llevar al usuario en lugar de simplemente bloquear sin explicación.

---

## 📁 Paso 4 — Public Guard (protege rutas públicas)

**Archivo:** `src/app/core/guards/public.guard.ts`

```typescript
import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';
import { AuthService } from '../services/auth.service';

export const publicGuard: CanActivateFn = () => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.isAuthenticated()) {
    return router.createUrlTree(['/pokemon']);
  }

  return true;
};
```

> 💡 **Es el inverso del `authGuard`:**
>
> | Guard | Pregunta | Usuario logueado | Usuario no logueado |
> |---|---|---|---|
> | `authGuard` | ¿estás logueado? | deja pasar ✅ | manda a /login ❌ |
> | `publicGuard` | ¿estás logueado? | manda a /pokemon ❌ | deja pasar ✅ |
>
> Sin `publicGuard`, un usuario ya logueado podría volver a `/login` y hacer login de nuevo — innecesario y confuso.

---

## 📁 Paso 5 — Configuración de Rutas

**Archivo:** `src/app/app.routes.ts`

```typescript
import { Routes } from '@angular/router';
import { authGuard } from './core/guards/auth.guard';
import { publicGuard } from './core/guards/public.guard';

export const routes: Routes = [
  {
    path: '',
    redirectTo: 'login',
    pathMatch: 'full'
  },
  {
    path: 'login',
    canActivate: [publicGuard],
    loadComponent: () =>
      import('./features/auth/pages/login/login').then(m => m.LoginPage)
  },
  {
    path: 'pokemon',
    canActivate: [authGuard],
    loadComponent: () =>
      import('./features/pokemon/pages/pokemon-list/pokemon-list')
        .then(m => m.PokemonListPage)
  },
  {
    path: '**',
    redirectTo: 'pokemon'
  }
];
```

### Explicación de cada ruta

> 💡 **`pathMatch: 'full'`** en la ruta raíz — la URL debe ser exactamente `''` para aplicar la redirección. Sin esto Angular podría redirigir en loop infinito.
>
> **`loadComponent`** — lazy loading. El código del componente no se descarga hasta que el usuario navega a esa ruta. En apps grandes con muchas páginas esto mejora drásticamente el tiempo de carga inicial.
>
> **`path: '**'`** — wildcard. Captura cualquier URL que no coincida con las anteriores. Siempre debe ser la **última ruta**. Redirige a `/pokemon` en lugar de `/login` porque si el usuario está logueado y escribe una URL inexistente, tiene más sentido mandarlo a la app que al login — si no está logueado, el `authGuard` de `/pokemon` lo redirigirá a `/login` automáticamente.

### Comportamiento resultante

| Escenario | Resultado |
|---|---|
| Sin login → intenta ir a `/pokemon` | → redirige a `/login` |
| Sin login → intenta ir a `/url-rara` | → intenta `/pokemon` → redirige a `/login` |
| Con login → intenta ir a `/login` | → redirige a `/pokemon` |
| Con login → intenta ir a `/url-rara` | → redirige a `/pokemon` |

---

## 📁 Paso 6 — Interceptor HTTP

**Archivo:** `src/app/core/interceptors/auth.interceptor.ts`

```typescript
import { HttpInterceptorFn } from '@angular/common/http';
import { inject } from '@angular/core';
import { AuthService } from '../services/auth.service';

export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  const user = authService.currentUser();

  const isFakeStoreAPI = req.url.includes('fakestoreapi.com');

  if (user?.token && isFakeStoreAPI) {
    const authReq = req.clone({
      setHeaders: {
        Authorization: `Bearer ${user.token}`
      }
    });
    return next(authReq);
  }

  return next(req);
};
```

### ¿Por qué el interceptor NO adjunta el token al login?

> 💡 El endpoint de login es `https://fakestoreapi.com/auth/login`. Sí pertenece a FakeStore (`isFakeStoreAPI = true`), pero la condición también verifica `user?.token`. Cuando el usuario hace login **no tiene token todavía** — `currentUser()` es `null`. Por eso la condición falla y la petición de login sale sin token. Exactamente lo correcto — no puedes autenticarte con un token que aún no tienes.

### Ejemplos de configuración del interceptor

#### Caso 1 — Interceptar TODAS las peticiones (cualquier API)
```typescript
// Sin filtro de dominio — agrega el token a absolutamente todo
if (user?.token) {
  const authReq = req.clone({
    setHeaders: { Authorization: `Bearer ${user.token}` }
  });
  return next(authReq);
}
```
> ⚠️ **No recomendado** — envía el token JWT a APIs de terceros como PokéAPI, exponiendo datos del usuario innecesariamente.

#### Caso 2 — Interceptar solo FakeStore (implementación actual)
```typescript
const isFakeStoreAPI = req.url.includes('fakestoreapi.com');

if (user?.token && isFakeStoreAPI) {
  // agrega token solo a FakeStore
}
```

#### Caso 3 — Interceptar múltiples APIs propias
```typescript
// Si en el futuro agregas tu propio backend además de FakeStore
const TRUSTED_APIS = [
  'fakestoreapi.com',
  'mi-propio-backend.com',
  'otra-api-de-confianza.com'
];

const isTrustedAPI = TRUSTED_APIS.some(domain => req.url.includes(domain));

if (user?.token && isTrustedAPI) {
  const authReq = req.clone({
    setHeaders: { Authorization: `Bearer ${user.token}` }
  });
  return next(authReq);
}
```

> 💡 **Principio de mínimo privilegio:** Solo comparte el token con APIs que realmente lo necesitan y en las que confías. PokéAPI no necesita tu token — enviárselo es una filtración de datos innecesaria.

### Registrar el interceptor en `app.config.ts`

```typescript
import { ApplicationConfig, provideBrowserGlobalErrorListeners } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient, withFetch, withInterceptors } from '@angular/common/http';
import { authInterceptor } from './core/interceptors/auth.interceptor';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideBrowserGlobalErrorListeners(),
    provideRouter(routes),
    provideHttpClient(
      withFetch(),
      withInterceptors([authInterceptor])
    ),
  ]
};
```

> 💡 **`withInterceptors([...])`** recibe un array — puedes registrar múltiples interceptores. Se ejecutan en orden de izquierda a derecha. Por ejemplo podrías tener:
> ```typescript
> withInterceptors([authInterceptor, errorInterceptor, loggingInterceptor])
> ```

---

## 📁 Paso 7 — Página de Login con Signals y Validaciones

**Archivo:** `src/app/features/auth/pages/login/login.ts`

```typescript
import { Component, computed, inject, signal } from '@angular/core';
import { Router } from '@angular/router';
import { FormsModule } from '@angular/forms';
import { AuthService } from '../../../../core/services/auth.service';
import { LoginRequest } from '../../../../core/models/auth.model';

@Component({
  selector: 'app-login',
  imports: [FormsModule],
  templateUrl: './login.html',
  styleUrl: './login.css'
})
export class LoginPage {

  private readonly authService = inject(AuthService);
  private readonly router = inject(Router);

  // --- Signals del formulario ---
  readonly username = signal('mor_2314');
  readonly password = signal('83r5^_');
  readonly isLoading = signal(false);
  readonly errorMessage = signal('');

  // --- Signals de interacción ---
  readonly usernameTouched = signal(false);
  readonly passwordTouched = signal(false);

  // --- Computed de validación ---
  readonly usernameError = computed(() => {
    if (!this.usernameTouched()) return '';
    if (!this.username()) return 'El usuario es requerido';
    return '';
  });

  readonly passwordError = computed(() => {
    if (!this.passwordTouched()) return '';
    if (!this.password()) return 'La contraseña es requerida';
    return '';
  });

  readonly isFormValid = computed(() =>
    !!this.username() && !!this.password()
  );

  onSubmit(): void {
    this.isLoading.set(true);
    this.errorMessage.set('');

    const credentials: LoginRequest = {
      username: this.username(),
      password: this.password()
    };

    this.authService.login(credentials).subscribe({
      next: () => {
        this.router.navigate(['/pokemon']);
      },
      error: () => {
        this.isLoading.set(false);
        this.errorMessage.set('Usuario o contraseña incorrectos');
      }
    });
  }
}
```

### Patrón "touched + error" para validaciones

> 💡 Este patrón evita mostrar errores antes de que el usuario interactúe con el formulario:
>
> ```
> Campo sin tocar    → no muestra error (usernameTouched = false)
> Campo tocado vacío → muestra error (usernameTouched = true + username = '')
> Campo tocado lleno → sin error (usernameTouched = true + username = 'valor')
> ```
>
> Es mejor UX mostrar errores solo cuando el usuario ha demostrado intención de llenar el campo.

### Por qué `isFormValid` no depende de los errores

```typescript
// ❌ Redundante — los errores ya verifican si hay valor
readonly isFormValid = computed(() =>
  !this.usernameError() &&
  !this.passwordError() &&
  !!this.username() &&    // ← ya lo verifica usernameError
  !!this.password()       // ← ya lo verifica passwordError
);

// ✅ Correcto — isFormValid responde solo "¿hay datos para enviar?"
readonly isFormValid = computed(() =>
  !!this.username() && !!this.password()
);
```

> 💡 Cada computed tiene **una sola responsabilidad**:
> - `usernameError` → ¿qué error mostrar en el campo?
> - `passwordError` → ¿qué error mostrar en el campo?
> - `isFormValid` → ¿se puede hacer submit?

### Por qué no hay validación manual en `onSubmit()`

```typescript
// ❌ Innecesario — el botón ya está deshabilitado con [disabled]="!isFormValid()"
onSubmit(): void {
  if (!this.username() || !this.password()) {
    this.errorMessage.set('Por favor completa todos los campos');
    return;
  }
  // ...
}
```

> 💡 **Principio:** No duplicar validaciones. Si el HTML previene el submit con `[disabled]`, validar lo mismo en TypeScript es código muerto — nunca se ejecutará. Duplicar lógica es una fuente de bugs futura.

---

**Archivo:** `src/app/features/auth/pages/login/login.html`

```html
<div class="min-h-screen bg-gray-100 flex items-center justify-center p-4">
  <div class="bg-white rounded-2xl shadow-lg w-full max-w-md p-8">

    <!-- Logo y título -->
    <div class="text-center mb-8">
      <div class="text-6xl mb-4">🐾</div>
      <h1 class="text-3xl font-bold text-gray-800">Pokédex</h1>
      <p class="text-gray-500 mt-2">Inicia sesión para continuar</p>
    </div>

    <!-- Mensaje de error global -->
    @if (errorMessage()) {
      <div class="bg-red-50 border border-red-200 text-red-600 rounded-lg p-3 mb-6 text-sm">
        {{ errorMessage() }}
      </div>
    }

    <!-- Formulario -->
    <form (ngSubmit)="onSubmit()">

      <!-- Username -->
      <div class="mb-4">
        <label class="block text-sm font-medium text-gray-700 mb-2">
          Usuario
        </label>
        <input
          type="text"
          [value]="username()"
          (input)="username.set($any($event.target).value)"
          (blur)="usernameTouched.set(true)"
          class="w-full border border-gray-300 rounded-lg px-4 py-3 text-gray-800 focus:outline-none focus:ring-2 focus:ring-blue-500"
          placeholder="Ingresa tu usuario"
        />
        <p class="text-red-500 text-sm mt-1 h-4">
          {{ usernameError() }}
        </p>
      </div>

      <!-- Password -->
      <div class="mb-4">
        <label class="block text-sm font-medium text-gray-700 mb-2">
          Contraseña
        </label>
        <input
          type="password"
          [value]="password()"
          (input)="password.set($any($event.target).value)"
          (blur)="passwordTouched.set(true)"
          class="w-full border border-gray-300 rounded-lg px-4 py-3 text-gray-800 focus:outline-none focus:ring-2 focus:ring-blue-500"
          placeholder="Ingresa tu contraseña"
        />
        <p class="text-red-500 text-sm mt-1 h-4">
          {{ passwordError() }}
        </p>
      </div>

      <!-- Botón -->
      <button
        type="submit"
        [disabled]="isLoading() || !isFormValid()"
        class="w-full bg-blue-600 hover:bg-blue-700 disabled:bg-blue-300 text-white font-semibold py-3 rounded-lg transition-colors duration-200">
        @if (isLoading()) {
          Iniciando sesión...
        } @else {
          Iniciar sesión
        }
      </button>

    </form>

    <!-- Credenciales de prueba -->
    <div class="mt-6 p-4 bg-gray-50 rounded-lg text-sm text-gray-500">
      <p class="font-medium mb-1">Credenciales de prueba:</p>
      <p>Usuario: <span class="font-mono text-gray-700">mor_2314</span></p>
      <p>Contraseña: <span class="font-mono text-gray-700">83r5^_</span></p>
    </div>

  </div>
</div>
```

### Conceptos del HTML

#### Binding bidireccional con Signals
```html
<input
  [value]="username()"
  (input)="username.set($any($event.target).value)"
  (blur)="usernameTouched.set(true)"
/>
```
> 💡 Dos eventos conectados al Signal:
> - **`[value]`** → del Signal al HTML (cuando el Signal cambia, el input se actualiza)
> - **`(input)`** → del HTML al Signal (cuando el usuario escribe, el Signal se actualiza)
> - **`(blur)`** → cuando el campo pierde el foco, marca como tocado

#### `@if` — nueva sintaxis Angular 17+
```html
@if (errorMessage()) {
  <div>{{ errorMessage() }}</div>
}
```
> 💡 Reemplaza al antiguo `*ngIf`. Solo renderiza el elemento si la condición es verdadera. Un string vacío es `false` — el error solo aparece cuando tiene contenido.

#### Espacio reservado para errores — evitar layout shift
```html
<p class="text-red-500 text-sm mt-1 h-4">
  {{ usernameError() }}
</p>
```
> 💡 La clase `h-4` reserva altura fija siempre, aunque el párrafo esté vacío. Cuando el error aparece ocupa ese espacio en lugar de empujar el contenido hacia abajo. Esto elimina el **layout shift** — el salto brusco que ocurre cuando aparecen elementos dinámicos.

---

## 📌 Estado final de la Fase 4

| Archivo | Estado |
|---|---|
| `core/models/auth.model.ts` | ✅ |
| `core/services/auth.service.ts` | ✅ |
| `core/guards/auth.guard.ts` | ✅ |
| `core/guards/public.guard.ts` | ✅ |
| `core/interceptors/auth.interceptor.ts` | ✅ |
| `app.routes.ts` | ✅ |
| `app.config.ts` | ✅ |
| `features/auth/pages/login/login.ts` | ✅ |
| `features/auth/pages/login/login.html` | ✅ |

---

## ⏭️ Siguiente paso

Con la autenticación completa, el siguiente paso es construir la feature principal:
- Crear el servicio de Pokémon que consume PokéAPI
- Construir el listado de Pokémon con paginación
- Construir el detalle de un Pokémon
- Aplicar Signals para el estado de la lista
- Agregar búsqueda con debounce

> 🎓 **Este archivo no requiere más modificaciones.** Todo el sistema de autenticación está documentado aquí.
