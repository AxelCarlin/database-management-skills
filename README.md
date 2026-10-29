# Database Management Skills

Colección de skills para diseño, administración y análisis de bases de datos.

Cada skill vive en su propia carpeta dentro de `skills/` y sigue el formato de un archivo `SKILL.md` con frontmatter YAML.

## Skills incluidas

### `dbml-sqlserver-schema-admin`

Diseña, normaliza, renombra y corrige esquemas de base de datos escritos en DBML para SQL Server.

Convenciones principales:

- Entidades en `UpperCamelCase` y singular.
- Llave primaria `Id`.
- Llaves foráneas con prefijo `Id`.
- Columnas de auditoría obligatorias.
- Orden fijo de columnas.
- Tablas de catálogo para campos categóricos.
- Tercera forma normal.
- Módulos y secciones claramente separados.

Esta variante deja `ModifiedAt` e `IdModifiedBy` como `NULL` hasta la primera modificación del registro.

### `dbml-sqlserver-schema-detective`

La misma base de diseño DBML/SQL Server, pero con otra política de auditoría:

- `ModifiedAt` e `IdModifiedBy` son obligatorios desde la creación.
- Al crear, `IdModifiedBy` toma el valor de `IdCreatedBy`.

> [!IMPORTANT]
> Ambas skills declaran el mismo `name: dbml-sqlserver-schema`. Instala o activa solo una variante a la vez para evitar colisiones.

## Estructura

```text
database-management-skills/
├── README.md
└── skills/
    ├── dbml-sqlserver-schema-admin/
    │   ├── SKILL.md
    │   └── references/
    │       └── example.dbml
    └── dbml-sqlserver-schema-detective/
        ├── SKILL.md
        └── references/
            └── example.dbml
```

## Uso

1. Elige la variante que corresponda a la política de auditoría de tu proyecto.
2. Copia su carpeta al directorio de skills de tu agente o herramienta.
3. Mantén el archivo `SKILL.md` y la carpeta `references/` juntos.

## Alcance

SQL Server es la única parte específica de la plataforma: tipos de datos y valores por defecto. Las reglas de nombres, normalización, auditoría y organización son agnósticas al dominio.
