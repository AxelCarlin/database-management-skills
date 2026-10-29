# Database Management Skills

A collection of skills for designing, administering, and reviewing databases.

Each skill lives in its own folder under `skills/` and follows the `SKILL.md` format with YAML front matter.

## Design target: SQL Server and .NET

These skills were designed primarily for **SQL Server** and **.NET** projects, specifically **.NET 10**. Their data type mappings, audit columns, nullability policies, naming conventions, and examples are optimized for that stack.

They can still work with other database engines and runtimes, but some definitions may need adaptation. For example, SQL Server types such as `datetime2` and `tinyint`, defaults such as `SYSUTCDATETIME()`, and .NET mappings such as `DateTime` and `byte` are specific to this target stack.

## Included skills

### `dbml-sqlserver-schema-admin`

Designs, normalizes, renames, and corrects database schemas written in DBML for SQL Server.

Main conventions:

- `UpperCamelCase` singular entity names.
- `Id` primary keys.
- Foreign keys prefixed with `Id`.
- Mandatory audit columns.
- Fixed column order.
- Lookup tables for categorical fields.
- Third Normal Form.
- Clearly separated modules and sections.

This variant keeps `ModifiedAt` and `IdModifiedBy` as `NULL` until the record is first modified.

### `dbml-sqlserver-schema-detective`

The same DBML/SQL Server design baseline, but with a different audit policy:

- `ModifiedAt` and `IdModifiedBy` are mandatory from creation.
- On insert, `IdModifiedBy` takes the value of `IdCreatedBy`.

> [!IMPORTANT]
> Both skills declare the same `name: dbml-sqlserver-schema`. Install or enable only one variant at a time to avoid collisions.

## Structure

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

## Usage

1. Choose the variant that matches your project's audit policy.
2. Copy its folder into the skills directory used by your agent or tool.
3. Keep the `SKILL.md` file and the `references/` folder together.

## Scope

The naming, normalization, audit, and organization rules are domain-agnostic. The platform-specific parts are the SQL Server data types and defaults and the .NET type mappings. Adapt those parts when using another engine or runtime.
