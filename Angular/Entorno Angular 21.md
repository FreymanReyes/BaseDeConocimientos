# 🛠️ Fase 1 — Instalación del Entorno y Creación del Proyecto

> **Proyecto:** `pokedex-ng` — Angular 21 + PokéAPI + FakeStore API (login) + Vitest
> **Sistema Operativo:** Windows
> **Editor:** VS Code

---

## 🎯 ¿Qué vamos a construir?

Una aplicación Angular 21 profesional con dos grandes features:

| Feature | API utilizada | Qué aprenderás |
|---|---|---|
| 🔐 Login + Rutas protegidas | FakeStore API | JWT, Guards, Interceptores HTTP, Seguridad |
| 🐾 Explorador Pokémon | PokéAPI | Consumo REST, paginación, búsqueda, caché con Signals |

---

## 📋 Herramientas necesarias

| Herramienta | Versión mínima | Función |
|---|---|---|
| Git | cualquier reciente | Control de versiones |
| NVM | 1.x | Gestor de versiones de Node |
| Node.js | v20+ (recomendado v22) | Motor de ejecución para Angular CLI |
| npm | v9+ | Gestor de paquetes |
| Angular CLI | v21 | Crear y gestionar proyectos Angular |
| GitHub Desktop | cualquier reciente | Interfaz visual para Git y GitHub |

---

## ✅ Paso 0 — Verificación inicial

Antes de instalar cualquier cosa, verificar qué está disponible abriendo PowerShell como Administrador y ejecutando:

```powershell
node --version
npm --version
ng version
git --version
```

> ⚠️ **PowerShell como Administrador:** Clic derecho en el ícono de PowerShell → *Ejecutar como administrador*. Algunos comandos de instalación requieren permisos elevados en Windows.

---

## 🔧 Paso 1 — Verificar Git

```powershell
git --version
```

**Resultado obtenido:**
```
git version 2.45.1.windows.1  ✅
```

> Git viene de forma separada en Windows. Si no está instalado, descargarlo desde https://git-scm.com/download/win

---

## 🔧 Paso 2 — Instalar NVM para Windows

### ¿Por qué NVM y no Node directamente?

Instalar Node.js directamente en el sistema es problemático porque:
- Distintos proyectos pueden requerir distintas versiones de Node
- Actualizar o cambiar versiones manualmente es tedioso y propenso a errores
- Desinstalar y reinstalar Node rompe configuraciones existentes

**NVM (Node Version Manager)** permite instalar múltiples versiones de Node y cambiar entre ellas con un solo comando.

> 💡 **Analogía:** NVM es como tener varios juegos de herramientas y poder elegir cuál usar según el trabajo, sin tener que desechar los demás.

### Instalación

1. Ir a: https://github.com/coreybutler/nvm-windows/releases/latest
2. Descargar el archivo **`nvm-setup.exe`** desde la sección *Assets*
3. Ejecutarlo como **Administrador** (clic derecho → *Ejecutar como administrador*)
4. Aceptar todas las opciones por defecto y finalizar

### Verificar instalación

> ⚠️ **Importante:** Cerrar PowerShell completamente y reabrirlo como Administrador antes de continuar. Windows necesita reiniciar la sesión de terminal para reconocer los nuevos comandos instalados.

```powershell
nvm version
```

**Resultado obtenido:**
```
1.2.2  ✅
```

---

## 🔧 Paso 3 — Instalar Node.js v22 con NVM

### ¿Por qué Node.js v22?

Angular 21 requiere Node v20 como mínimo. La v22 es la versión **LTS (Long Term Support)** más reciente, lo que significa que tendrá soporte y parches de seguridad por varios años. Siempre se recomienda usar la versión LTS en proyectos reales.

### Instalar la versión 22

```powershell
nvm install 22
```

**Resultado obtenido:**
```
Downloading node.js version 22.22.1 (64-bit)...
Extracting node and npm...
Complete
Installation complete.
If you want to use this version, type:

nvm use 22.22.1
```

### Activar la versión instalada

```powershell
nvm use 22
```

> 💡 **Diferencia entre `install` y `use`:** NVM permite tener varias versiones instaladas simultáneamente. `install` descarga la versión al sistema, pero `use` es lo que la activa como versión actual. Sin este paso, Node no estará disponible aunque esté instalado.

**Resultado obtenido:**
```
Now using node v22.22.1 (64-bit)  ✅
```

---

## 🔧 Paso 4 — Verificar Node.js y npm

```powershell
node --version
npm --version
```

**Resultados obtenidos:**
```
v22.22.1   ✅
10.9.4     ✅  (versión inicial incluida con Node — se actualizará en el paso 6)
```

> npm viene incluido automáticamente con Node.js. No es necesario instalarlo por separado.

---

## 🔧 Paso 5 — Instalar Angular CLI v21

### ¿Qué es Angular CLI?

Angular CLI (Command Line Interface) es la herramienta oficial de Angular para crear y gestionar proyectos desde la terminal. Con un solo comando puedes crear proyectos, componentes, servicios, guards, y más, sin crear archivos a mano.

```powershell
npm install -g @angular/cli@21
```

> 💡 **¿Qué significa el flag `-g`?** Significa *global*. Instala Angular CLI de forma global en el sistema, haciendo que el comando `ng` esté disponible desde cualquier carpeta, no solo en un proyecto específico.
>
> **Analogía:** Es como instalar un programa en Windows para todos los usuarios, en lugar de solo para una carpeta específica.

**Resultado obtenido:**
```
added 285 packages in 31s  ✅

npm notice
npm notice New major version of npm available! 10.9.4 -> 11.12.0
npm notice Changelog: https://github.com/npm/cli/releases/tag/v11.12.0
npm notice To update run: npm install -g npm@11.12.0
npm notice
```

> npm notifica que hay una versión más reciente disponible. Se actualiza en el siguiente paso.

---

## 🔧 Paso 6 — Actualizar npm a la versión más reciente

npm avisa que existe una nueva versión mayor. Se actualiza para tener las últimas mejoras de seguridad y rendimiento desde el inicio del proyecto.

```powershell
npm install -g npm@11.12.0
```

**Resultado obtenido:**
```
removed 73 packages, and changed 104 packages in 12s  ✅
```

---

## 🔧 Paso 7 — Verificación final del entorno completo

```powershell
ng version
npm --version
```

**Resultados obtenidos:**
```
     _                      _                 ____ _     ___
    / \   _ __   __ _ _   _| | __ _ _ __     / ___| |   |_ _|
   / △ \ | '_ \ / _` | | | | |/ _` | '__|   | |   | |    | |
  / ___ \| | | | (_| | |_| | | (_| | |      | |___| |___ | |
 /_/   \_\_| |_|\__, |\__,_|_|\__,_|_|       \____|_____|___|
                |___/

Angular CLI : 21.2.3   ✅
Node.js     : 22.22.1  ✅
npm         : 11.12.0  ✅
OS          : win32 x64
```

---

## 🏗️ Paso 8 — Crear el proyecto Angular 21

### ¿Dónde guardar el proyecto?

Se recomienda crear una carpeta dedicada para proyectos de desarrollo fuera de Documentos o el Escritorio, ya que esas carpetas suelen sincronizarse con OneDrive en Windows, lo que puede causar conflictos con `node_modules` (que contiene miles de archivos pequeños).

> En este caso el proyecto se creó directamente en la carpeta de GitHub Desktop: `C:\Users\pc\documents\GitHub`

```powershell
cd C:\Users\pc\documents\GitHub
ng new nombreDelProyecto
```

### Preguntas durante la creación — respondidas así:

**Pregunta 1 — Datos de uso anónimos:**
```
Would you like to share pseudonymous usage data about this project
with the Angular Team at Google under Google's Privacy Policy?
→ N (No)
```
> Google solicita permiso para recolectar datos anónimos de uso del CLI. No es obligatorio y no afecta el proyecto. En entornos profesionales siempre se declina.

**Pregunta 2 — Sistema de estilos:**
```
√ Which stylesheet system would you like to use?
→ CSS
```
> Se elige CSS plano porque el proyecto usará **TailwindCSS**, que funciona perfectamente con CSS sin necesidad de SCSS. Agregar SCSS sería complejidad innecesaria en esta etapa.

**Pregunta 3 — Server-Side Rendering (SSR) y SSG:**
```
√ Do you want to enable Server-Side Rendering (SSR) and
  Static Site Generation (SSG/Prerendering)?
→ No
```
> SSR renderiza las páginas en el servidor antes de enviarlas al navegador, lo que mejora el SEO. Para este proyecto de aprendizaje de frontend puro no es necesario. Se puede agregar después si se requiere.

**Pregunta 4 — Herramientas de IA (nueva en Angular 21):**
```
? Which AI tools do you want to configure with Angular best practices?
>(*) None
```
> Angular 21 permite configurar archivos de contexto para asistentes de IA como Copilot, Cursor o Claude. Se deja en *None* por ahora y se puede configurar manualmente después si se desea.

**Resultado obtenido — proyecto creado exitosamente:**
```
✔ Packages installed successfully.
    Successfully initialized git.
```

---

## 🔧 Paso 9 — Abrir el proyecto y ejecutarlo

### Entrar a la carpeta del proyecto
```powershell
cd nombreDelProyecto
```

### Abrir en VS Code
```powershell
code .
```
> El punto `.` significa "la carpeta actual". Este comando le dice a VS Code que abra el proyecto completo en esa ubicación.

### Ejecutar el servidor de desarrollo
```powershell
ng serve
```
> `ng serve` compila el proyecto y levanta un servidor local. Cada vez que se guarda un archivo, la app se recarga automáticamente en el navegador sin necesidad de reiniciar nada.

### Abrir en el navegador
```
http://localhost:4200
```

**Resultado:** Pantalla de bienvenida de Angular corriendo correctamente. ✅

---

## 📌 Estado final — Entorno y proyecto listos ✅

| Herramienta | Versión instalada | Estado |
|---|---|---|
| Git | 2.45.1 | ✅ Listo |
| NVM | 1.2.2 | ✅ Listo |
| Node.js | v22.22.1 | ✅ Listo |
| npm | 11.12.0 | ✅ Listo |
| Angular CLI | 21.2.3 | ✅ Listo |
| GitHub Desktop | instalado con cuenta activa | ✅ Listo |
| Proyecto `nombreDelProyecto` | Angular 21.2.3 | ✅ Corriendo en localhost:4200 |

---

> 🎓 **Este archivo no requiere más modificaciones.**
> El entorno está 100% configurado y el proyecto base está creado y corriendo.
> A partir de aquí comienza el desarrollo: arquitectura de carpetas, consumo de APIs, Signals, rutas protegidas y tests.
