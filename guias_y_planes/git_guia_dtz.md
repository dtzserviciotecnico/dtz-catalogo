# 🗂️ Guía de Git — dtz-catalogo
> Control de versiones para el proyecto DTZ. Windows + PowerShell.

---

## 1. Instalación (una sola vez)

Descargá Git desde: **https://git-scm.com/download/win**

Durante la instalación dejá todas las opciones por defecto.

Verificá que quedó instalado abriendo PowerShell y escribiendo:
```powershell
git --version
# Debería mostrar algo como: git version 2.45.0
```

---

## 2. Configuración inicial (una sola vez)

```powershell
git config --global user.name "Lucas Miranda"
git config --global user.email "tu-email@gmail.com"
```

> Usá el mismo email que tenés en GitHub. Esto firma cada commit con tu nombre.

---

## 3. Conectar tu carpeta local con el repo de GitHub

Tu proyecto ya existe en GitHub. Tenés que "clonar" o "vincular" la carpeta local.

### Opción A — Ya tenés la carpeta `WebDTZ` con archivos (tu caso)

```powershell
# Posicionarte en la carpeta del proyecto
cd C:\Users\TIAGO\Documents\ERP\WebDTZ

# Inicializar Git en esa carpeta
git init

# Conectarla al repo remoto de GitHub
git remote add origin https://github.com/lucasmirandau/dtz-catalogo.git

# Traer el historial que ya está en GitHub
git fetch origin

# Vincular tu rama local con la remota
git branch -M main
git branch --set-upstream-to=origin/main main

# Traer los últimos cambios
git pull origin main --allow-unrelated-histories
```

### Opción B — Clonar desde cero (si no tenés la carpeta)
```powershell
git clone https://github.com/lucasmirandau/dtz-catalogo.git
cd dtz-catalogo
```

---

## 4. El flujo de trabajo diario

Cada vez que editás un archivo, el proceso es siempre el mismo:

```
Editás un archivo → git add → git commit → git push
```

### Paso a paso con comandos:

```powershell
# 1. Ver qué archivos cambiaron
git status

# 2. Agregar los archivos al "paquete" que vas a guardar
git add admin.html          # un archivo específico
git add index.html          # otro archivo
git add .                   # todos los archivos cambiados a la vez

# 3. Crear el punto de guardado con un mensaje descriptivo
git commit -m "Agrego tab de usuarios y forgot password"

# 4. Subir a GitHub
git push origin main
```

---

## 5. Ver el historial de versiones

```powershell
# Ver todos los commits (puntos de guardado)
git log --oneline

# Ejemplo de lo que verías:
# a3f9c12 Agrego carousel de banners al index
# b72e401 Agrego tab de usuarios y forgot password
# c91d883 Primer commit — catálogo inicial
```

Cada línea es un punto al que podés volver.

---

## 6. Volver atrás — los tres escenarios

### Escenario A — Todavía no hiciste commit (editaste pero no guardaste)
```powershell
# Descartar los cambios en un archivo y volver a como estaba
git checkout -- admin.html

# Descartar TODOS los cambios sin commitear
git checkout -- .
```
⚠️ Esto es irreversible. Los cambios se pierden.

---

### Escenario B — Ya hiciste commit pero no hiciste push
```powershell
# Ver el historial
git log --oneline

# Deshacer el último commit (los cambios vuelven a "sin commitear")
git reset --soft HEAD~1

# Deshacer el último commit Y los cambios del archivo también
git reset --hard HEAD~1
```

---

### Escenario C — Ya hiciste push a GitHub (el caso más común)

```powershell
# Ver el historial y copiar el código del commit al que querés volver
git log --oneline

# Ejemplo: querés volver al commit b72e401
# Opción segura: crear un nuevo commit que deshace los cambios
git revert a3f9c12

# Eso crea un commit nuevo que revierte los cambios → luego pusheas normal
git push origin main
```

> ✅ `git revert` es la opción más segura cuando ya subiste a GitHub,
> porque no borra historia — agrega un commit que deshace el anterior.

---

### Escenario D — Ver cómo estaba un archivo en una versión anterior
```powershell
# Sin tocar nada, solo ver el contenido de admin.html en el commit b72e401
git show b72e401:admin.html

# Restaurar ese archivo al estado de ese commit (sin commitear todavía)
git checkout b72e401 -- admin.html
```

---

## 7. Cheatsheet — comandos más usados

| Comando | ¿Para qué sirve? |
|---|---|
| `git status` | Ver qué archivos cambiaron |
| `git add .` | Preparar todos los cambios |
| `git commit -m "mensaje"` | Crear punto de guardado |
| `git push origin main` | Subir a GitHub |
| `git pull origin main` | Bajar cambios de GitHub |
| `git log --oneline` | Ver historial de commits |
| `git checkout -- archivo` | Descartar cambios sin commitear |
| `git revert HASH` | Deshacer un commit ya pusheado |
| `git diff` | Ver exactamente qué líneas cambiaron |

---

## 8. Buenas prácticas para los mensajes de commit

Un buen mensaje explica **qué cambiaste y por qué**, no solo "cambios":

```
❌ Malo:   "cambios", "fix", "update", "arreglos"

✅ Bueno:  "Agrego tab de usuarios con gestión de roles"
✅ Bueno:  "Fix: el login no mostraba mensaje de error correctamente"
✅ Bueno:  "Carousel de banners — 3 slides, lazy loading activado"
✅ Bueno:  "Agrego .gitignore para excluir archivos SQL"
```

---

## 9. Flujo recomendado antes de que yo edite archivos

Antes de que empieces una sesión de trabajo conmigo, hacé siempre:

```powershell
# 1. Asegurarte de tener la última versión
git pull origin main

# 2. (Opcional pero recomendado) Crear un punto de guardado del estado actual
git add .
git commit -m "Backup antes de sesión — estado estable"
git push origin main
```

Así si algo sale mal durante una sesión, siempre podés volver a ese punto.

---

## 10. Configuración recomendada para PowerShell (alias útiles)

Podés agregar estos atajos a tu perfil de PowerShell para ir más rápido:

```powershell
# Abrir el perfil de PowerShell
notepad $PROFILE

# Pegar estas líneas al final:
function gs { git status }
function ga { git add . }
function gp { git push origin main }
function gl { git log --oneline -10 }
function gpl { git pull origin main }
```

Después de guardar, reiniciá PowerShell y podés escribir simplemente `gs`, `ga`, `gp`, etc.

---

*Guía específica para el proyecto dtz-catalogo — Lucas Miranda*
