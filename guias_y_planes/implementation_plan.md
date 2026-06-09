# Plan: Roles, Overrides de Productos y Promociones — dtz-catalogo

## Descripción general

Tres mejoras interrelacionadas:
1. **Roles profesionales** — 3 niveles con permisos diferenciados
2. **Overrides de productos** — editar nombre y precio sin tocar el import de Odoo
3. **Promociones por volumen** — descuento automático en el carrito según cantidad

---

## 1. Redefinición de Roles

### Estructura propuesta

| Rol | Nombre técnico | Quién lo usa | Qué puede hacer |
|---|---|---|---|
| Superadministrador | `superadmin` | Desarrollador (Lucas) | Todo: usuarios, roles, productos, promociones |
| Administrador | `admin` | Dueño / Encargado | Productos (overrides, import), promociones. NO gestión de usuarios |
| Empleado | `staff` | Personal de la empresa | Solo imágenes de productos |

### Cambios en la UI del tab Usuarios

- El chip del usuario logueado muestra su rol visible (`SUPERADMIN`, `ADMIN`, `STAFF`)
- Solo `superadmin` ve el formulario de crear usuario y la lista completa
- Solo `superadmin` puede cambiar roles (y no puede degradarse a sí mismo)
- El selector de rol al crear usuario muestra las 3 opciones solo para `superadmin`

### Cambios en el acceso por tab

| Tab | superadmin | admin | staff |
|---|---|---|---|
| 📊 Importar | ✅ | ✅ | ❌ oculto |
| 📦 Productos | ✅ (todo) | ✅ (overrides + imágenes) | 🟡 solo imágenes |
| 🏷️ Promociones | ✅ | ✅ | ❌ oculto |
| 🖼 Logo | ✅ | ✅ | ❌ oculto |
| 👥 Usuarios | ✅ | ❌ oculto | ❌ oculto |

---

## 2. Overrides de nombre y precio

### El problema
Odoo exporta nombre y precio. Cada importación **reemplaza todos los productos**.
Si el admin edita un nombre o precio directamente en `productos`, el próximo import lo pisa.

### La solución: tabla separada `product_overrides`

Los datos de Odoo van a `productos` (como siempre).
Los ajustes manuales van a `product_overrides`, clave: el `code` del producto.

Al mostrar en el catálogo público:
```
nombre mostrado = product_overrides.display_name ?? productos.name
precio mostrado = product_overrides.custom_price  ?? productos.price
```

El import de Odoo **nunca toca** `product_overrides`.

### SQL nuevo

```sql
CREATE TABLE IF NOT EXISTS public.product_overrides (
  code          TEXT PRIMARY KEY,
  display_name  TEXT,        -- nombre alternativo para mostrar
  custom_price  NUMERIC,     -- precio web (ej: promo, precio especial)
  updated_by    TEXT,        -- email del admin que lo editó
  updated_at    TIMESTAMPTZ  DEFAULT now()
);

ALTER TABLE public.product_overrides ENABLE ROW LEVEL SECURITY;

-- Lectura pública (el index.html lo necesita)
CREATE POLICY "Public read overrides"
  ON public.product_overrides FOR SELECT USING (true);

-- Solo autenticados pueden escribir
CREATE POLICY "Auth write overrides"
  ON public.product_overrides FOR ALL
  TO authenticated USING (true) WITH CHECK (true);
```

### UI en el tab Productos (admin + superadmin)

- Nueva columna "Override" en la tabla de productos
- Botón ✏️ en cada fila que abre un mini-modal:
  - Campo: **Nombre web** (placeholder: nombre actual de Odoo)
  - Campo: **Precio web** (placeholder: precio actual de Odoo)
  - Botón Guardar / Limpiar override
- Si tiene override activo → badge visible `✏ Editado`
- Si limpiás el override → vuelve a mostrar los datos de Odoo

---

## 3. Promociones por volumen

### Diseño elegido: múltiples niveles (más flexible que un solo umbral)

El admin puede crear varias reglas. Ejemplo real:

| Regla | Cantidad mínima en carrito | Descuento |
|---|---|---|
| Promo Pequeña | 3 productos | 5% |
| Promo Mediana | 6 productos | 10% |
| Promo Grande | 10 productos | 15% |

En el carrito se aplica automáticamente **el nivel más alto** que corresponda.

> **¿Por qué no un solo umbral?** Porque esto permite incentivar compras progresivas
> ("si agregás 2 más, te hacemos 10% en vez de 5%").

### SQL nuevo

```sql
CREATE TABLE IF NOT EXISTS public.promotions (
  id               SERIAL PRIMARY KEY,
  nombre           TEXT NOT NULL,
  min_cantidad     INTEGER NOT NULL CHECK (min_cantidad > 0),
  descuento_pct    NUMERIC(5,2) NOT NULL CHECK (descuento_pct > 0 AND descuento_pct <= 100),
  activo           BOOLEAN DEFAULT true,
  created_at       TIMESTAMPTZ DEFAULT now()
);

ALTER TABLE public.promotions ENABLE ROW LEVEL SECURITY;

-- Lectura pública (el index.html la necesita para aplicar descuentos)
CREATE POLICY "Public read promotions"
  ON public.promotions FOR SELECT USING (true);

-- Solo autenticados pueden gestionar
CREATE POLICY "Auth manage promotions"
  ON public.promotions FOR ALL
  TO authenticated USING (true) WITH CHECK (true);
```

### UI: nuevo tab 🏷️ Promociones (admin + superadmin)

**Panel izquierdo — Crear promoción:**
- Campo: Nombre (ej: "Promo Volumen Mayo")
- Campo: Cantidad mínima de productos en carrito
- Campo: % de descuento
- Toggle: Activo / Inactivo
- Botón: Crear

**Panel derecho — Promociones activas:**
- Tabla con nombre, cantidad mínima, descuento, estado
- Botón activar/desactivar por fila
- Botón eliminar

### UI en index.html — El carrito

Cuando el total de ítems en el carrito alcanza un umbral:
- Se muestra un banner verde en el carrito:
  ```
  🏷️ Promo Volumen: 10% de descuento aplicado (6+ productos)
  ```
- El subtotal muestra el precio original tachado y el precio con descuento
- Si el cliente está cerca del siguiente nivel:
  ```
  💡 Agregá 2 productos más y obtenés 15% de descuento
  ```

---

## Archivos a modificar

### [MODIFY] admin.html
- Actualizar roles (`superadmin`, `admin`, `staff`)
- Control de acceso por tab según rol
- Nuevo tab 🏷️ Promociones
- Columna override en tab Productos con mini-modal de edición
- UI del tab Usuarios actualizada para 3 roles

### [MODIFY] index.html
- Carga de `product_overrides` al iniciar (junto con productos)
- Carga de `promotions` activas al iniciar
- Lógica del carrito: aplicar descuento + mostrar banner + sugerencia de próximo nivel

### [NEW] SQL a ejecutar en Supabase (local, no al repo)
- `product_overrides` table + RLS
- `promotions` table + RLS
- Update constraint de `profiles.role` para incluir `superadmin` y `staff`

---

## Preguntas abiertas

> [!IMPORTANT]
> **¿Cómo querés manejar el precio en el carrito cuando hay override?**
> - Opción A: El precio del override reemplaza el de Odoo completamente (simple)
> - Opción B: El override es un "precio promocional web" que se muestra junto al precio original tachado (más visual)

> [!IMPORTANT]
> **¿Las promociones aplican sobre el precio final (con override) o sobre el precio de Odoo?**
> Lo lógico sería sobre el precio que el cliente ve (es decir, con override si existe).

> [!NOTE]
> **Sobre el rol `superadmin`:** Tu usuario actual en Supabase tiene rol `admin`
> (creado por el trigger). Antes de ejecutar los cambios, habrá que actualizarlo
> a `superadmin` manualmente en la tabla `profiles`.

---

## Verificación

1. Login con cada tipo de usuario → verificar que solo ve sus tabs
2. Import de Odoo → verificar que no pisa overrides existentes
3. Crear promo → agregar productos al carrito → verificar descuento aplicado
4. Próximo nivel sugerido → verificar mensaje correcto
5. Git commit limpio con todos los cambios

