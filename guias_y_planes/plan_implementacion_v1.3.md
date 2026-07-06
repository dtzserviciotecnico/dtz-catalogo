# Plan de Implementación v1.3.0 — Cupones de Descuento (RPC) y Precios Rápidos de Taller

Este plan detalla la arquitectura técnica para implementar el **Sistema de Cupones de Descuento (con seguridad bancaria vía RPC en Supabase)** y el **Módulo de Tarifario Rápido y Autocompletado** en el sistema de servicio técnico.

---

## 1. 🎟️ Sistema de Descuentos / Cupones (Web & Admin)

### A. Base de Datos & Seguridad (Supabase RPC)
Para cumplir con **La Solución de Nivel Dios** y evitar que cualquier persona técnica pueda listar los cupones desde la consola del navegador:
1. **Tabla `descuentos`**:
   - `id` (serial / int, PK)
   - `codigo` (text, único, normalizado en mayúsculas sin espacios, ej: `PRIMERANIVERSARIO`)
   - `nombre` (text, descripción, ej: "Primer año del local 2984")
   - `porcentaje` (numeric/int, ej: `25` para 25%)
   - `activo` (boolean, default `true`)
   - `created_at` (timestamptz)
2. **Políticas RLS**:
   - `SELECT`, `INSERT`, `UPDATE`, `DELETE`: **Denegado para el rol público (`anon`)**. Solo accesible mediante token de usuarios autenticados con rol `admin` o `superadmin` en el panel.
3. **Función Secreta en Base de Datos (RPC)**:
   - Crear función `validar_cupon(codigo_input text)` en PostgreSQL (con `SECURITY DEFINER`).
   - Esta función busca en silencio en la tabla `descuentos` donde `codigo = upper(trim(codigo_input))` y `activo = true`.
   - Si existe, devuelve JSON con `{ "valido": true, "porcentaje": 25, "nombre": "..." }`.
   - Si no existe, devuelve `{ "valido": false }`. Es matemáticamente imposible listar otros cupones.

### B. Panel de Administración (`admin.html`)
1. **Nueva Solapa en Navegación**: `🎟️ Descuentos` (ubicada junto a Promociones o Banners).
2. **Gestor Completo (CRUD)**:
   - Formulario de creación rápida con conversión automática del código a mayúsculas (`.toUpperCase()`).
   - Tabla con listado de cupones creados mostrando código, descripción, porcentaje, interruptor de pausa/activación (`⚡` / `⏸️`) y botón de eliminación (`🗑️`).

### C. Carrito de Compras (`index.html`)
1. **Input de Descuento**:
   - Ubicado sobre la selección de sucursal de retiro en el cajón del carrito.
   - Campo de texto y botón `[ APLICAR ]`.
2. **Lógica Comercial (No Acumulable)**:
   - Al aplicar un cupón válido mediante el llamado a la función RPC `validar_cupon`, se recalculará el total del carrito.
   - **Regla de Negocio**: Si el cliente ya tenía un descuento por volumen aplicado, el sistema compara ambos y aplica automáticamente **el descuento que represente un mayor ahorro para el cliente** (o reemplaza la promo por volumen por el cupón, evitando sumar ambos porcentajes para proteger el margen de ganancia).
3. **Integración WhatsApp**:
   - El desglose enviado por WhatsApp mostrará claramente la aplicación del cupón y el monto ahorrado.

---

## 2. 🛠️ Tarifario de Precios Rápidos & Autocompletado (`reparaciones.html`)

### A. Base de Datos
1. **Tabla `precios_taller`**:
   - `id` (serial / int, PK)
   - `categoria` (text, ej: `SAM / MOTO / REDMI`, `IPHONE`, `TABLETS`, `VARIOS`)
   - `servicio` (text, ej: `Limpieza Pin de Carga`, `Cambio de Placa de carga`)
   - `precio` (numeric/int, ej: `20000`)
   - `orden` (int, para ordenamiento de presentación)
2. **Políticas RLS**:
   - `SELECT`: Público para usuarios autenticados (`staff`, `admin`, `superadmin`).
   - `INSERT`, `UPDATE`, `DELETE`: Bloqueado exclusivamente para roles `admin` y `superadmin`.

### B. Interfaz y Experiencia (UX Nivel Dios 🌟)
1. **Modal de Tarifario Rápido (`💲 Precios`)**:
   - Botón en la barra superior junto a "+ Nueva" y "📜 Auditoría".
   - Al abrir, muestra tarjetas organizadas por categoría para rápida lectura del staff.
   - Si el usuario es `Admin` o `Superadmin`, mostrará en línea los botones de edición rápida (`✏️`), agregar servicio y eliminar.
2. **Super-Poder de Autocompletado en Formulario (`+ Nueva`)**:
   - Junto al campo **Presupuesto Total ($)**, se incorporará un botón o selector rápido: `[ ⚡ Cargar Precio de Lista ▾ ]`.
   - Al seleccionar un servicio de la lista desplegable (ej: `IPHONE - Cambio de Placa de carga ($60.000)`):
     - El campo `Presupuesto` se autocompletará al instante con `60000`.
     - El campo `Trabajo Realizado / Fallas` insertará automáticamente el texto del servicio.
     - Reduciendo el tiempo de recepción en mostrador a menos de 10 segundos.

---

## 3. Guía de Ejecución Paso a Paso

1. **Paso 1**: Crear archivo SQL de migración (`scripts_sql/08_cupones_y_tarifario_v1.3.sql`) con las nuevas tablas, políticas RLS y la función secreta `validar_cupon`.
2. **Paso 2**: Implementar la solapa de Descuentos en `admin.html`.
3. **Paso 3**: Integrar input, llamada RPC y lógica no acumulable en el carrito y WhatsApp en `index.html`.
4. **Paso 4**: Implementar el modal de Tarifario y el selector rápido de autocompletado en `reparaciones.html`.
5. **Paso 5**: Verificar y actualizar la documentación técnica y el manual del desarrollador en la carpeta `guias_y_planes/`.

---

## Verificación del Sistema
- Probar ingreso de cupón secreto válido e inválido en consola para verificar impermeabilidad del RPC.
- Probar creación y borrado de precios de taller desde rol `Admin` y verificar que el rol `Staff` solo pueda leer.
