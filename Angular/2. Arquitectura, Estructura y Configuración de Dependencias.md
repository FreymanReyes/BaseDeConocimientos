# 🏗️ Arquitectura, Estructura y Configuración de Dependencias

> **Proyecto:** `nombreDelProyecto` — Angular 21 + PokéAPI + FakeStore API (login) + Vitest
> **Prerequisito:** Tener completada la Fase 1 (entorno instalado y proyecto creado corriendo en localhost:4200)

---

## 📌 Cambio importante en Angular 17+

Antes de continuar, hay una diferencia clave entre Angular 16 y versiones anteriores vs Angular 17 en adelante.

A partir de **Angular 17** se eliminó el sufijo `.component` de los archivos:

| Antes (Angular 16 y anterior) | Ahora (Angular 17+) |
|---|---|
| `app.component.ts` | `app.ts` |
| `app.component.html` | `app.html` |
| `app.component.css` | `app.css` |
| `app.component.spec.ts` | `app.spec.ts` |

> 💡 **¿Por qué cambiaron esto?** Para simplificar. En proyectos grandes escribir `.component` en cada archivo era repetitivo. Angular asume que si está dentro de una carpeta de componente, es un componente. El código interno lo deja claro.

La estructura real dentro de `src/app` al crear el proyecto es:

```
src/
└── app/
    ├── app.ts
    ├── app.html
    ├── app.css
    ├── app.spec.ts
    ├── app.routes.ts
    └── app.config.ts
```

---

## 🏛️ Fase 2 — Arquitectura de Carpetas

### ¿Por qué necesitamos una arquitectura?

Los proyectos que crecen sin estructura se vuelven imposibles de mantener. Una buena arquitectura desde el día 1 permite:
- Escalar el proyecto sin caos
- Que cualquier desarrollador entienda dónde está cada cosa
- Eliminar features sin romper otras partes
- Reutilizar código de forma ordenada

### La arquitectura elegida — Feature-based

Se llama **arquitectura por features** y es el estándar en equipos profesionales:

> **Cada cosa en su lugar. Cada lugar con un propósito.**

---

### 📁 Explicación de cada carpeta

#### `core/` — El corazón de la app
Todo lo que existe **una sola vez** en toda la app.

```
core/
├── guards/         → Protegen rutas (¿está el usuario logueado?)
├── interceptors/   → Modifican peticiones HTTP (agregan el token JWT)
├── models/         → Las interfaces TypeScript (¿cómo luce un Pokémon?)
└── services/       → La lógica de negocio (llamadas a la API)
```

> 💡 **Regla de oro:** Todo lo que va en `core/` se instancia una sola vez. Los servicios de autenticación, interceptores HTTP, guards — existen una vez y son usados por toda la app.
>
> **Analogía:** Es como el sistema eléctrico de una casa. No pones un transformador en cada habitación — hay uno central que alimenta todo.

---

#### `features/` — Las páginas de la app
Cada feature es **autocontenida** — todo lo que necesita está dentro de su carpeta.

```
features/
├── auth/           → Todo lo relacionado con login/logout
│   ├── pages/      → El componente de la página de login
│   └── components/ → Componentes pequeños usados solo en auth
└── pokemon/        → Todo lo relacionado con Pokémon
    ├── pages/      → Lista de pokémon, detalle de un pokémon
    └── components/ → Tarjeta de pokémon, buscador, etc.
```

> 💡 **¿Por qué separar por features?** Si mañana decides eliminar la sección de Pokémon, simplemente borras la carpeta `pokemon/` — nada está mezclado con otras partes.
>
> **Analogía:** Es como los departamentos de una empresa. Ventas no mezcla sus archivos con los de Contabilidad.

---

#### `shared/` — Lo que se comparte
Solo lo que se usa en **más de un feature**.

```
shared/
├── components/     → Componentes reutilizables (botón, spinner, card)
└── pipes/          → Transformadores de datos (capitalizar texto, etc.)
```

> 💡 **Regla práctica:** Si te preguntas "¿esto lo usaré en otro lugar?", la respuesta define si va en `shared/` o en el feature específico.

---

#### `layout/` — La estructura visual
La estructura fija de la app que envuelve todo el contenido.

```
layout/
├── navbar/         → La barra de navegación superior
└── footer/         → El pie de página
```

> 💡 El navbar y el footer no son un "feature" ni son "compartidos" en el sentido tradicional — son la estructura permanente de la app. Tienen su propia carpeta para dejar claro su rol.

---

### 🗺️ Estructura completa final

```
src/
└── app/
    ├── core/
    │   ├── guards/
    │   ├── interceptors/
    │   ├── models/
    │   └── services/
    ├── features/
    │   ├── auth/
    │   │   ├── pages/
    │   │   └── components/
    │   └── pokemon/
    │       ├── pages/
    │       └── components/
    ├── shared/
    │   ├── components/
    │   └── pipes/
    └── layout/
        ├── navbar/
        └── footer/
```

---

### ⚡ Comando para crear toda la estructura de una vez

Desde la raíz del proyecto en PowerShell:

```powershell
mkdir src/app/core/guards, src/app/core/interceptors, src/app/core/models, src/app/core/services, src/app/features/auth/pages, src/app/features/auth/components, src/app/features/pokemon/pages, src/app/features/pokemon/components, src/app/shared/components, src/app/shared/pipes, src/app/layout/navbar, src/app/layout/footer
```

> 💡 **Nota sobre carpetas vacías y Git:** Git no trackea carpetas vacías. No es necesario crear archivos `.gitkeep` — en cuanto se cree el primer archivo real dentro de cada carpeta, Git la reconocerá automáticamente en el siguiente commit.

---

## 📦 Fase 3 — Instalación y Configuración de Dependencias

### ¿Qué instalamos y por qué?

| Dependencia | Tipo | Para qué |
|---|---|---|
| TailwindCSS v3 | devDependency | Estilos utilitarios |
| PostCSS | devDependency | Procesador CSS requerido por Tailwind |
| Autoprefixer | devDependency | Prefijos de navegador automáticos |
| Vitest | devDependency | Framework moderno de tests |
| HttpClient | built-in Angular | Peticiones HTTP a las APIs |

---

### 🎨 TailwindCSS

#### ¿Qué es TailwindCSS?

Es un framework de CSS utilitario. En lugar de escribir clases CSS propias, usas clases predefinidas directamente en el HTML.

**Sin Tailwind:**
```css
.mi-boton {
  background-color: blue;
  padding: 8px 16px;
  border-radius: 4px;
  color: white;
}
```
```html
<button class="mi-boton">Click</button>
```

**Con Tailwind:**
```html
<button class="bg-blue-500 px-4 py-2 rounded text-white">Click</button>
```

#### ¿Qué pasa con Tailwind en producción?

Tailwind es una herramienta de construcción, no una librería de runtime:

```
Desarrollo:
Tailwind incluye TODAS las clases disponibles → CSS ~3MB (para desarrollo rápido)

Producción:
Tailwind escanea SOLO las clases que usaste → CSS ~5KB a ~50KB (optimizado)
```

> 💡 Este proceso de eliminar clases no usadas se llama **purging** o **tree-shaking**. El usuario final nunca descarga Tailwind — solo descarga el CSS puro y optimizado resultante.

---

#### ⚠️ Tailwind v4 vs v3 con Angular

Tailwind v4 es muy reciente y su integración con Angular CLI tiene inconsistencias al momento de escribir esta guía.

**Lo que NO funciona con Tailwind v4 + Angular 21:**
- Crear un archivo `vite.config.ts` con el plugin de Tailwind v4

> 💡 **¿Qué es Vite?** Es el bundler (empaquetador) que Angular 21 usa internamente para compilar el proyecto. Toma todos los archivos separados (TypeScript, HTML, CSS) y los empaqueta en archivos optimizados para el navegador.
>
> Aunque Angular usa Vite internamente, **no expone** un `vite.config.ts` directamente — Angular tiene su propio sistema de configuración en `angular.json`. Crear un `vite.config.ts` manual no tiene efecto en un proyecto Angular CLI.

**La solución: usar Tailwind v3** que tiene soporte oficial y estable con Angular.

---

#### Instalación de TailwindCSS v3

##### Paso 1 — Instalar dependencias

```powershell
npm install -D tailwindcss@3 postcss autoprefixer
```

> 💡 El flag `-D` significa `--save-dev`. Estas dependencias son solo para desarrollo — en producción solo queda el CSS generado, no las herramientas que lo generaron.

**Resultado esperado:**
```
added 60 packages, and audited 529 packages in 9s  ✅
```

##### Paso 2 — Generar archivo de configuración

```powershell
npx tailwindcss init
```

**Resultado esperado:**
```
Created Tailwind CSS config file: tailwind.config.js  ✅
```

##### Paso 3 — Configurar `tailwind.config.js`

Abrir el archivo generado y reemplazar con:

```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./src/**/*.{html,ts}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

> 💡 **`content`** le dice a Tailwind dónde buscar las clases que usas. Sin esto no sabe qué clases generar y el CSS sale vacío.

##### Paso 4 — Configurar `src/styles.css`

Reemplazar todo el contenido con:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

> 💡 Tailwind v3 divide su CSS en tres capas:
> - `base` → Reset de estilos del navegador
> - `components` → Clases de componentes reutilizables
> - `utilities` → Las clases utilitarias (`bg-blue-500`, `text-white`, etc.)

##### Paso 5 — Crear `postcss.config.js`

```powershell
New-Item postcss.config.js -ItemType File
```

Pegar dentro del archivo:

```javascript
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

> 💡 **¿Por qué este archivo?** Angular usa PostCSS para procesar el CSS. Este archivo le dice a PostCSS que use Tailwind y Autoprefixer como plugins. Es el puente entre Angular y Tailwind v3.

##### Paso 6 — Verificar que funciona

```powershell
ng serve
```

Abrir `src/app/app.html` y reemplazar el contenido con:

```html
<div class="min-h-screen bg-blue-500 flex items-center justify-center">
  <h1 class="text-4xl font-bold text-white">
    Tailwind funciona ✅
  </h1>
</div>
```

**Resultado esperado:** Fondo azul con texto blanco grande en `http://localhost:4200` ✅

---

### 🔧 HttpClient — Configuración para llamadas HTTP

#### ¿Qué es HttpClient?

Es el módulo oficial de Angular para hacer peticiones HTTP a APIs externas. Sin él no es posible llamar a PokéAPI ni a FakeStore API.

> 💡 En Angular 21 nada viene preconfigurado por defecto — cada módulo se activa explícitamente. Esto hace la app más liviana porque solo carga lo que realmente usa.

#### Instalación

HttpClient viene incluido con Angular — no requiere `npm install`. Solo necesita configurarse en `src/app/app.config.ts`:

```typescript
import { ApplicationConfig, provideZoneChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient, withFetch } from '@angular/common/http';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true }),
    provideRouter(routes),
    provideHttpClient(withFetch()),
  ]
};
```

> 💡 **¿Qué es `withFetch()`?** Le dice a Angular que use la API nativa `fetch` del navegador en lugar de `XMLHttpRequest` (la forma antigua). `fetch` es más moderna, más eficiente y es el estándar actual.

---

### 🧪 Vitest — Tests modernos

#### ¿Por qué Vitest y no Karma?

Angular genera proyectos con **Karma + Jasmine** por defecto. Pero:

| | Karma (default) | Vitest |
|---|---|---|
| Creado en | 2012 | 2021 |
| Velocidad | Lento | Muy rápido |
| Integración TypeScript | Limitada | Nativa |
| Uso en industria | Decayendo | Creciendo |

> 💡 Usar Vitest es como cambiar un auto 2012 por uno 2024 — mismo destino, mejor experiencia.

#### Instalación

##### Paso 1 — Instalar dependencias

```powershell
npm install -D vitest @vitest/coverage-v8 happy-dom @analogjs/vitest-angular
```

> 💡 **¿Qué es cada cosa?**
> - `vitest` → El framework de tests
> - `@vitest/coverage-v8` → Genera reportes de cobertura (qué % del código está testeado)
> - `happy-dom` → Simula el DOM del navegador para los tests (más rápido que jsdom)
> - `@analogjs/vitest-angular` → El puente entre Vitest y Angular

##### Paso 2 — Crear `vitest.config.ts` en la raíz

```powershell
New-Item vitest.config.ts -ItemType File
```

```typescript
import { defineConfig } from 'vitest/config';
import angular from '@analogjs/vitest-angular/plugin';

export default defineConfig({
  plugins: [angular()],
  test: {
    globals: true,
    environment: 'happy-dom',
    include: ['src/**/*.spec.ts'],
    reporters: ['verbose'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html'],
      include: ['src/app/**/*.ts'],
      exclude: ['src/app/**/*.spec.ts', 'src/main.ts'],
    },
  },
});
```

##### Paso 3 — Agregar scripts en `package.json`

Abrir `package.json` y dentro de `"scripts"` agregar:

```json
"test": "vitest",
"test:coverage": "vitest run --coverage"
```

> 💡 **¿Qué hace cada script?**
> - `npm test` → Corre los tests en modo watch (se re-ejecutan al guardar)
> - `npm run test:coverage` → Corre los tests una vez y genera reporte de cobertura

---

## 📌 Estado final de dependencias configuradas

| Herramienta | Versión | Estado |
|---|---|---|
| TailwindCSS | v3 | ✅ |
| PostCSS | latest | ✅ |
| Autoprefixer | latest | ✅ |
| HttpClient | built-in Angular | ✅ |
| Vitest | latest | ✅ |

---

## 🗂️ Archivos nuevos en la raíz del proyecto

```
nombreDelProyecto/
├── tailwind.config.js    ← Configuración de Tailwind (qué archivos escanear)
├── postcss.config.js     ← Puente entre Angular y Tailwind
└── vitest.config.ts      ← Configuración del framework de tests
```

---

## ⏭️ Siguiente paso

Con el entorno completamente preparado, comienza el desarrollo real:
- Crear los componentes de layout (navbar, footer)
- Implementar el sistema de rutas
- Construir el módulo de autenticación con FakeStore API
- Construir el explorador Pokémon con PokéAPI
- Aplicar Signals en toda la app
- Escribir tests con Vitest

> 🎓 **Este archivo no requiere más modificaciones.** Toda la configuración base del proyecto está documentada aquí.
