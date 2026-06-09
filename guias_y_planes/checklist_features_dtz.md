# ✅ Checklist de Funcionalidades — dtz-catalogo

---

## 1. 👤 Sistema de Usuarios Administradores

### Decisión de diseño: ¿Email o Apodo?

> **Recomendación:** Usar **email + contraseña** con Supabase Auth.
> Supabase Auth está diseñado nativamente para emails. Usar apodos requeriría una tabla custom y más lógica manual.
> Se puede mostrar el apodo como "nombre visible" aunque internamente el login sea por email.

### Sub-opciones evaluadas

- [ ] **Opción A (Recomendada):** Email + contraseña con Supabase Auth
  - Login con email institucional o personal
  - Se le asigna un "nombre de display" (apodo) visible en el panel
  - Supabase gestiona sesiones, tokens y expiración automáticamente
  - Se puede habilitar 2FA en el futuro

- [ ] **Opción B:** Nombre/apodo + contraseña (tabla custom)
  - Requiere tabla `admins` con columnas: `username`, `password_hash`, `role`
  - Hay que implementar el hash de contraseñas manualmente (bcrypt)
  - Mayor riesgo de errores de seguridad
  - ❌ NO recomendado salvo exigencia explícita del cliente

### Tareas de implementación (Opción A)

- [ ] Activar **Supabase Auth** en el proyecto (ya puede estar activo)
- [ ] Crear tabla `profiles` vinculada a `auth.users`:
  - `id` (uuid, FK a auth.users)
  - `display_name` (text) — el apodo visible
  - `role` (text: `admin` | `viewer`)
  - `created_at` (timestamp)
- [ ] Crear función en Supabase para auto-crear perfil al registrar usuario
- [ ] Proteger `admin.html` con verificación de sesión activa al cargar la página
- [ ] Implementar logout en el panel admin
- [ ] Implementar pantalla de login (`login.html`) separada del admin
- [ ] (Opcional) Roles: admin puede crear/editar/eliminar; viewer solo puede ver

---

## 2. 🖼️ Banner Carousel en index.html

### Decisión de diseño

> **Prioridad:** El catálogo de productos. El carousel debe ser liviano, rápido y no bloquear la carga de los productos.

### Especificaciones técnicas del carousel

- [ ] **Cantidad de imágenes:** 3 a 5 slides (configurable desde admin)
- [ ] **Medidas recomendadas para las imágenes:**
  - Ancho: **1280px** (mínimo) — 1920px (ideal para full-width)
  - Alto: **400px a 480px** (relación 16:4 aprox.)
  - Formato: **WebP** preferido (más liviano); aceptar JPG/PNG como fallback
  - Peso máximo por imagen: **150 KB** (optimizada)
- [ ] Mostrar en el panel admin las medidas exactas al momento de subir imagen
- [ ] Lazy loading activado en las imágenes del carousel
- [ ] Autoplay opcional (pausar al hover)
- [ ] Navegación con flechas y dots indicadores
- [ ] Responsive (adaptarse a mobile sin deformarse)
- [ ] Transición CSS suave (no usar librerías pesadas como Swiper.js salvo necesidad)

### Tabla en Supabase para el carousel

- [ ] Crear tabla `banners`:
  - `id` (int, PK)
  - `image_url` (text) — URL de imagen en Supabase Storage
  - `titulo` (text, nullable) — texto superpuesto opcional
  - `subtitulo` (text, nullable)
  - `link_url` (text, nullable) — si el banner es clickeable
  - `orden` (int) — posición en el carousel
  - `activo` (boolean) — para activar/desactivar sin borrar
  - `created_at` (timestamp)

### Panel Admin — gestión de banners

- [ ] Sección "Banners / Publicidades" en admin.html
- [ ] Mostrar medidas recomendadas al lado del botón de subida
- [ ] Preview de la imagen antes de confirmar
- [ ] Control de orden (drag & drop o flechas arriba/abajo)
- [ ] Toggle activo/inactivo por slide
- [ ] Límite máximo de 5 banners activos simultáneos

---

## 3. 📄 Documentación (ver archivo `documentacion_dtz.txt`)

- [ ] Redactar sección para el dueño de DTZ
- [ ] Redactar sección para empleados que usen el admin
- [ ] Redactar sección para el desarrollador (yo)
- [ ] Redactar sección sobre columnas de la base de datos y resiliencia ante cambios de Odoo

---

## 🔐 Seguridad transversal (aplica a todo)

- [ ] Supabase RLS (Row Level Security) activado en todas las tablas
- [ ] `config.js` solo expone la `anon key` (nunca la `service_role key`)
- [ ] `admin.html` redirige a `login.html` si no hay sesión activa
- [ ] Storage de Supabase con políticas: solo usuarios autenticados pueden subir imágenes
- [ ] No hay datos sensibles hardcodeados en el frontend

---

## 5. 🏗️ Infraestructura y Despliegue

- [ ] Vincular dominio dtzserviciotecnico.com.ar en GitHub Pages y NIC.ar
- [ ] Actualizar URLs en Supabase Auth (Site URL y Redirect URLs)
- [ ] **Traspaso de Supabase:** Transferir propiedad del proyecto a la cuenta de DTZ (o invitar al dueño como Owner)

---

_Última actualización: 2026-06-09_
