# Diagrama Entidad-Relación - Inventario Herzab

## Diagrama ER Completo

```mermaid
erDiagram
    %% CATÁLOGOS / MAESTROS
    TIPO_SOLICITUD ||--o{ SOLICITUD : "define"
    PROVEEDOR ||--o{ SOLICITUD : "suministra"
    CATEGORIA ||--o{ MATERIAL : "clasifica"
    MARCA ||--o{ MATERIAL : "fabrica"
    UNIDAD ||--o{ MATERIAL : "medida"
    ALMACEN ||--o{ UBICACION : "contiene"
    OFICINA ||--o{ UBICACION : "localiza"
    
    %% TABLAS DE NEGOCIO
    MATERIAL ||--o{ STOCK : "inventariado"
    UBICACION ||--o{ STOCK : "ubicación"
    SOLICITUD ||--o{ DETALLE_SOLICITUD : "detalla"
    MATERIAL ||--o{ DETALLE_SOLICITUD : "solicita"
    STOCK ||--o{ MOVIMIENTO : "afecta"
    DETALLE_SOLICITUD ||--o{ MOVIMIENTO : "genera"
    
    %% SOLICITUD DE REGISTRO DE MATERIAL
    SOLICITUD_REGISTRO_MAT ||--o{ DETALLE_REGISTRO_MAT : "especifica"
    
    %% AUTENTICACIÓN Y AUTORIZACIÓN
    USUARIO ||--o{ SOLICITUD : "crea_solicitud"
    USUARIO ||--o{ SOLICITUD : "aprueba_solicitud"
    USUARIO ||--o{ SOLICITUD : "mueve_stock"
    USUARIO ||--o{ SOLICITUD_REGISTRO_MAT : "pide_registro"
    USUARIO ||--o{ SOLICITUD_REGISTRO_MAT : "atiende_registro"
    USUARIO ||--o{ USUARIO_ROL : "tiene"
    USUARIO ||--o{ USUARIO_OFICINA : "pertenece"
    ROL ||--o{ USUARIO_ROL : "asigna"
    ROL ||--o{ ROL_PERMISO : "otorga"
    PERMISO ||--o{ ROL_PERMISO : "define"
    OFICINA ||--o{ USUARIO_OFICINA : "acceso"
    
    %% MIGRACIONES
    MIGRACIONES ||--|| "CONTROL" : "registra"

    %% === ENTIDADES CON ATRIBUTOS ===
    
    TIPO_SOLICITUD {
        int id PK
        string nombre UK
    }
    
    PROVEEDOR {
        int id PK
        char ruc UK
        string razon_social
        string telefono
        string direccion
        boolean habilitado
    }
    
    CATEGORIA {
        int id PK
        string nombre UK
        boolean habilitado
    }
    
    MARCA {
        int id PK
        string nombre UK
        boolean habilitado
    }
    
    UNIDAD {
        int id PK
        string nombre UK
        boolean habilitado
    }
    
    ALMACEN {
        int id PK
        string nombre UK
        boolean habilitado
    }
    
    OFICINA {
        int id PK
        char codigo UK
        string nombre UK
        int numero_pisos
        string direccion
        boolean habilitado
    }
    
    MATERIAL {
        int id PK
        char codigo UK
        string descripcion
        int id_categoria FK
        int id_marca FK
        int id_unidad FK
        boolean habilitado
    }
    
    UBICACION {
        int id PK
        int id_oficina FK
        int id_almacen FK
        string nombre
        boolean habilitado
        unique "id_oficina+id_almacen+nombre"
    }
    
    SOLICITUD {
        int id PK
        char codigo UK
        timestamp fecha
        timestamp fecha_aprobacion
        int id_tipo_sol FK
        int id_proveedor FK
        string origen
        int idusuariosolicitante FK
        int idusuarioaprobador FK
        int idusuariomovimiento FK
        string destino
        string estado
        longblob pdf
        text observacion
    }
    
    DETALLE_SOLICITUD {
        int id PK
        int id_solicitud FK
        int id_material FK
        int cantidad_solicitada
    }
    
    STOCK {
        int id PK
        int id_material FK
        int id_ubicacion FK
        int cantidad_actual
        int stock_minimo
        string estado
        boolean habilitado
        unique "id_material+id_ubicacion"
    }
    
    MOVIMIENTO {
        int id PK
        char codigo UK
        int id_stock FK
        int id_detalle FK
        string estado
        string motivo
        timestamp fecha
    }
    
    SOLICITUD_REGISTRO_MAT {
        int id PK
        char codigo UK
        timestamp fecha
        string titulo
        int idusuariosolicitante FK
        int idusuarioaprobador FK
        string motivo
        string estado
        text observacion
    }
    
    DETALLE_REGISTRO_MAT {
        int id PK
        int id_registro FK
        string nombre
        string unidad_sugerida
        string categoria_sugerida
        string marca_sugerida
    }
    
    USUARIO {
        int id PK
        string nombre
        string email UK
        string password_hash
        string avatar_iniciales
        boolean habilitado
        datetime ultimo_login_at
        datetime created_at
        datetime updated_at
    }
    
    ROL {
        int id PK
        string codigo UK
        string nombre
        string descripcion
        boolean habilitado
        datetime created_at
        datetime updated_at
    }
    
    PERMISO {
        int id PK
        string codigo UK
        string nombre
        string descripcion
        string modulo
        boolean habilitado
        datetime created_at
        datetime updated_at
    }
    
    USUARIO_ROL {
        int user_id FK
        int role_id FK
        datetime created_at
        PK "user_id+role_id"
    }
    
    ROL_PERMISO {
        int role_id FK
        int permission_id FK
        datetime created_at
        PK "role_id+permission_id"
    }
    
    USUARIO_OFICINA {
        int user_id FK
        int oficina_id FK
        boolean es_default
        datetime created_at
        PK "user_id+oficina_id"
    }
    
    MIGRACIONES {
        int id PK
        string archivo UK
        datetime ejecutado_at
    }
```

---

## Descripción de Entidades Principales

### 📦 Módulo de Inventario

| Tabla | Descripción |
|-------|-------------|
| **MATERIAL** | Catálogo de artículos. Código único, descripción, categoría, marca, unidad. |
| **STOCK** | Cantidad actual de cada material en cada ubicación. Calcula estado automáticamente (SIN STOCK, DISPONIBLE, STOCK BAJO). |
| **UBICACION** | Lugar físico donde se almacena material (oficina + almacén + nombre). |
| **MOVIMIENTO** | Registro de entrada/salida de materiales. Vinculado a detalle de solicitud y stock. |

### 🔄 Módulo de Solicitudes

| Tabla | Descripción |
|-------|-------------|
| **SOLICITUD** | Solicitud de entrada/salida. Estados: Pendiente, Aprobada, Rechazada. Origen: normal o manual. |
| **DETALLE_SOLICITUD** | Líneas de una solicitud (qué material y cuánto). |
| **TIPO_SOLICITUD** | Catálogo: ENTRADA o SALIDA. |

### 📝 Módulo de Registro de Material

| Tabla | Descripción |
|-------|-------------|
| **SOLICITUD_REGISTRO_MAT** | Pedido de registro de material nuevo (aún no en catálogo). |
| **DETALLE_REGISTRO_MAT** | Detalles del material sugerido (nombre, unidad, categoría, marca sugeridas). |

### 🔐 Módulo de Autenticación y Autorización (RBAC)

| Tabla | Descripción |
|-------|-------------|
| **USUARIO** | Cuentas de acceso al sistema (email + contraseña hash). |
| **ROL** | Roles disponibles (SUPERADMIN, LOGISTICA, RRHH). |
| **PERMISO** | Permisos atómicos del sistema (ej: crear_solicitud, aprobar_solicitud). |
| **USUARIO_ROL** | Relación N:M usuario ↔ rol. |
| **ROL_PERMISO** | Relación N:M rol ↔ permiso. |
| **USUARIO_OFICINA** | Relación N:M usuario ↔ oficina (multi-oficina). |

### 📍 Catálogos/Maestros

| Tabla | Descripción |
|-------|-------------|
| **PROVEEDOR** | Empresas que abastecen materiales. |
| **CATEGORIA** | Clasificación de materiales. |
| **MARCA** | Fabricantes. |
| **UNIDAD** | Unidades de medida (Unidad, Caja, Paquete, Litro, etc.). |
| **ALMACEN** | Pisos/bodegas (ej: PISO 1). |
| **OFICINA** | Sucursales (ej: PIURA, LIMA). |

---

## Relaciones Clave

### Flujo de Solicitudes
```
USUARIO crea → SOLICITUD → DETALLE_SOLICITUD ↔ MATERIAL
                    ↓
            TIPO_SOLICITUD (ENTRADA/SALIDA)
                    ↓
            USUARIO aprueba
                    ↓
            MOVIMIENTO (afecta STOCK)
```

### Flujo de Registro de Material
```
USUARIO pide → SOLICITUD_REGISTRO_MAT → DETALLE_REGISTRO_MAT
                        ↓
                USUARIO atiende (admin)
```

### Estructura de Autorización
```
USUARIO → USUARIO_ROL → ROL → ROL_PERMISO → PERMISO
USUARIO → USUARIO_OFICINA → OFICINA (multi-oficina)
```

### Estructura de Inventario
```
MATERIAL → STOCK ← UBICACION
           ↑
         MOVIMIENTO ← DETALLE_SOLICITUD
```

---

## Triggers Automáticos

1. **trg_stock_estado_bi** (BEFORE INSERT en t_stock)
   - Calcula estado automáticamente basado en cantidad vs mínimo
   - SIN STOCK: cantidad = 0
   - STOCK BAJO: 0 < cantidad ≤ stock_minimo
   - DISPONIBLE: cantidad > stock_minimo

2. **trg_stock_estado_bu** (BEFORE UPDATE en t_stock)
   - Recalcula estado al actualizar cantidad/mínimo

---

## Vista Materializada

**v_inventario_completo**: Consulta el inventario completo con toda la información asociada (material, stock, ubicación, almacén, oficina).

```sql
SELECT stock_id, material_id, material_codigo, material_descripcion,
       categoria, marca, unidad, cantidad_actual, stock_minimo, estado,
       ubicacion, almacen, oficina, habilitado
FROM v_inventario_completo;
```

---

## Permisos del Sistema (16 total)

### Solicitudes (3)
- `crear_solicitud` — Crear solicitud de salida
- `crear_entrada` — Crear solicitud de entrada
- `aprobar_solicitud` — Aprobar/rechazar solicitudes
- `ver_solicitudes_propias` — Ver mis solicitudes
- `ver_solicitudes_todas` — Ver todas las solicitudes

### Registro de Material (4)
- `pedir_registro_material` — Pedir registro
- `atender_registro_material` — Atender registro
- `ver_registro_material_propio` — Ver mis pedidos
- `ver_registro_material_todos` — Ver todos los pedidos

### Inventario (2)
- `ver_inventario` — Consultar inventario
- `administrar_inventario` — Administrar inventario

### Administración (3)
- `gestionar_usuarios` — Gestionar usuarios
- `gestionar_roles` — Gestionar roles
- `ver_permisos` — Consultar permisos
- `gestionar_oficinas` — Gestionar oficinas

---

## Asignación de Permisos por Rol

| Rol | Permisos |
|-----|----------|
| **SUPERADMIN** | Todos (16) |
| **LOGISTICA** | crear_solicitud, ver_solicitudes_propias, ver_solicitudes_todas, aprobar_solicitud, pedir_registro_material, ver_registro_material_propio, ver_inventario (7) |
| **RRHH** | crear_solicitud, ver_solicitudes_propias, pedir_registro_material, ver_registro_material_propio (4) |

