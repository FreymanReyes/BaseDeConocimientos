# 🛠️ Fase 1 — Instalación del Entorno de Desarrollo

> **Proyecto:** Angular 21 + FakeStore API + Vitest  
> **Sistema Operativo:** Windows  
> **Editor:** VS Code

---

## 📋 Herramientas necesarias

| Herramienta | Versión mínima | Función |
|---|---|---|
| Git | cualquier reciente | Control de versiones |
| NVM | 1.x | Gestor de versiones de Node |
| Node.js | v20+ (recomendado v22) | Motor de ejecución para Angular CLI |
| npm | v9+ | Gestor de paquetes |
| Angular CLI | v21 | Crear y gestionar proyectos Angular |

---

## ✅ Verificación inicial

Antes de instalar cualquier cosa, verificar qué está disponible ejecutando en PowerShell:

```powershell
node --version
npm --version
ng version
git --version
```

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

**NVM (Node Version Manager)** permite instalar múltiples versiones de Node y cambiar entre ellas con un solo comando.

### Instalación

1. Ir a: https://github.com/coreybutler/nvm-windows/releases/latest
2. Descargar el archivo **`nvm-setup.exe`** desde la sección *Assets*
3. Ejecutarlo como **Administrador** (clic derecho → *Ejecutar como administrador*)
4. Aceptar todas las opciones por defecto y finalizar

### Verificar instalación

> ⚠️ **Importante:** Cerrar PowerShell completamente y reabrirlo como Administrador antes de continuar.

```powershell
nvm version
```

**Resultado obtenido:**
```
1.2.2  ✅
```

---

## 🔧 Paso 3 — Instalar Node.js v22 con NVM

### Instalar la versión 22

```powershell
nvm install 22
```

Este comando descarga e instala la versión LTS más reciente de Node 22.  
Angular 21 requiere Node v20 como mínimo; v22 es la versión recomendada.

**Resultado obtenido:**
```
Downloading node.js version 22.22.1 (64-bit)...
Extracting node and npm...
Complete
Installation complete.
```

### Activar la versión instalada

```powershell
nvm use 22
```

> 💡 **Diferencia entre `install` y `use`:** NVM permite tener varias versiones instaladas simultáneamente. `install` descarga la versión, `use` la activa como la versión actual del sistema.

**Resultado obtenido:**
```
Now using node v22.22.1 (64-bit)  ✅
```

---

## 🔧 Paso 4 — Verificar Node.js y npm inicial

```powershell
node --version
npm --version
```

**Resultados obtenidos:**
```
v22.22.1   ✅
10.9.4     ✅  (versión inicial incluida con Node)
```

---

## 🔧 Paso 5 — Instalar Angular CLI v21

```powershell
npm install -g @angular/cli@21
```

El flag `-g` instala Angular CLI de forma **global**, lo que hace que el comando `ng` esté disponible desde cualquier carpeta del sistema.

> 💡 **Analogía:** Es como instalar un programa en Windows para que cualquier usuario pueda usarlo, sin importar desde qué carpeta se ejecute.

**Resultado obtenido:**
```
added 285 packages in 31s  ✅
```

---

## 🔧 Paso 6 — Actualizar npm a la versión más reciente

Durante la instalación de Angular CLI, npm notificó que había una nueva versión mayor disponible. Se actualizó para tener las últimas mejoras de seguridad y rendimiento:

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
Angular CLI : 21.2.3   ✅
Node.js     : 22.22.1  ✅
npm         : 11.12.0  ✅
OS          : win32 x64
```

---

## 📌 Estado final del entorno — Fase 1 completa ✅

| Herramienta | Versión instalada | Estado |
|---|---|---|
| Git | 2.45.1 | ✅ Listo |
| NVM | 1.2.2 | ✅ Listo |
| Node.js | v22.22.1 | ✅ Listo |
| npm | 11.12.0 | ✅ Listo |
| Angular CLI | 21.2.3 | ✅ Listo |
| GitHub Desktop | instalado con cuenta activa | ✅ Listo |

> 🎓 **Este archivo no requiere más modificaciones.** El entorno de desarrollo está 100% configurado y listo para el desarrollo del proyecto.
