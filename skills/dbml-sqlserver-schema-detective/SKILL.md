---
name: dbml-sqlserver-schema
description: Designs, restructures, renames and normalizes database schemas written in DBML (dbdiagram.io) following a strict convention - UpperCamelCase singular entities, Id primary keys, Id-prefixed foreign keys, mandatory audit columns, fixed column order, lookup tables for categorical fields, Third Normal Form, and clearly flagged modules/sections. Data types target SQL Server; everything else is domain-agnostic. Use this skill whenever the user mentions DBML, dbdiagram, database schema, data model, tables, entities, columns, foreign keys, normalization, or pastes code containing "Table ... {", "Ref:" or "TableGroup", even if they do not explicitly ask for the skill.
---

# DBML Schema Design (SQL Server)

## Purpose
Take concepts, fields, or an existing DBML schema and return a complete, corrected schema, valid DBML ready to paste into dbdiagram.io. By default return **only DBML**. Add a final `## Observations` section only when there are serious inconsistencies or ambiguous decisions.

The skill is domain-agnostic. **SQL Server is the only platform-specific part** (data types and defaults, section 6). A full example lives in `references/example.dbml`.

---

## 1. Project header

```dbml
Project <ProjectName> {
  database_type: 'SQLServer'
  Note: '<One or two sentences: purpose, scope, main modules.>'
}
```

Directly below, a short conventions comment block (names, audit columns, `RecordStatus` meaning, UTC policy).

---

## 2. Naming rules

| Element | Rule | Correct | Incorrect |
|---|---|---|---|
| Entity | UpperCamelCase, **singular** | `PermissionType` | `PermissionTypes`, `permission_type` |
| Column | UpperCamelCase | `PostalCode` | `postal_code` |
| Primary key | Always `Id` | `Id` | `UserId`, `IdUser` |
| Foreign key | `Id` + referenced entity (or `Id` + role + entity) | `IdUserType`, `IdWarehouseOrigin` | `UserTypeId`, `UserType` |
| Timestamp (date + time) | Suffix `At`, type `datetime2` | `CreatedAt`, `LastLoginAt` | `CreationDate`, `DeliveryTime` |
| Calendar date (no time) | Suffix `Date`, type `date` | `BirthDate`, `DeliveryDate` | `Birthday` |
| Descriptive text | `Description` | `Product.Description` | `Product.Name` |
| Name of a person/user/account | **Only exception**: `Name` | `User.Name` | `User.Description` |
| Business boolean | Prefix `Is` / `Has`, or an adjective + type `bit` | `IsPrimary`, `EmailVerified` | `Flag1` |

Additional rules:
- ASCII only: no accents, spaces, hyphens, underscores or prefixes like `tbl`/`str`.
- Table and column names in English.
- Junction tables (N:M) concatenate both entities in singular: `UserRole`.
- Lookup tables: `<Concept>Type` or `<Concept>Category` (e.g. `UserType`, `SignalType`).

---

## 3. Mandatory columns in EVERY entity

| Column | DBML type | .NET type | Null | Purpose |
|---|---|---|---|---|
| `Id` | `int [pk, increment]` | `int` | no | Surrogate primary key |
| `CreatedAt` | `datetime2` | `DateTime` | no | Creation timestamp (default `SYSUTCDATETIME()`) |
| `ModifiedAt` | `datetime2` | `DateTime` | no | Last modification (default `SYSUTCDATETIME()`; update on every change) |
| `RecordStatus` | `tinyint` | `byte` | no | Technical lifecycle state (default 1) |
| `IdCreatedBy` | `int` | `int` | no | FK → `User.Id`, who created the row |
| `IdModifiedBy` | `int` | `int` | no | FK → `User.Id`, who last modified the row (equals creator at creation) |

Notes:
- `RecordStatus`: `tinyint` (maps to `byte`). Never `smallint` (maps to `short`). Standard values: `0 = inactive`, `1 = active`; document extra values in the column note. Use `bit` only if the project already does.
- Add the columns if missing; fix name/type if they exist with another shape.
- Bootstrap: the root `User` row is created by a controlled seed/migration that references itself.
- Alternative policy (only if the user asks): `ModifiedAt` and `IdModifiedBy` nullable until the first update.

---

## 4. Column order (required)

1. `Id`
2. Descriptive columns (`Name`, `Description`, business attributes, business flags)
3. Date columns (`CreatedAt`, `ModifiedAt`, then business dates/timestamps)
4. `RecordStatus`
5. Referential columns: `IdCreatedBy`, `IdModifiedBy`, then the remaining `Id*` FKs

Separate each block with a blank line. Example:

```
Id, Name, Street, PostalCode, Description,
CreatedAt, ModifiedAt, SubscribedAt,
RecordStatus,
IdCreatedBy, IdModifiedBy, IdUserType
```

---

## 5. Categorical fields → lookup tables

Analyze **every field** before adding it. If its values come from a finite, relatively stable set (user type, product code, unit of measure, category, workflow state, country, currency, role, shift…), do not store free text: create a lookup entity and an FK.

```dbml
Table UserType {
  Id int [pk, increment, note: '...']
  Description nvarchar(100) [not null, unique, note: '...']
  // CreatedAt, ModifiedAt, RecordStatus, IdCreatedBy, IdModifiedBy as in section 3
}
```

- Use `Description` with a `unique` constraint. Add an optional `Code varchar(30) [unique]` when code logic needs a stable key.
- Do NOT turn into lookups: `RecordStatus`, free text, proper names, measured numeric values, timestamps.

---

## 6. SQL Server data types (the only platform-specific section)

| Concept | DBML type |
|---|---|
| Id / FK | `int` (`bigint` for high-volume tables: logs, events, readings) |
| Human-readable text (names, descriptions) | `nvarchar(n)` |
| Technical text (codes, hashes, tokens, tags) | `varchar(n)` |
| Long text | `nvarchar(max)` |
| Email | `nvarchar(254)` |
| Password hash | `varchar(255)` |
| Timestamp | `datetime2` (store UTC; use `datetimeoffset` only if offsets must be preserved) |
| Calendar date | `date` |
| Boolean | `bit` |
| Status | `tinyint` |
| Money | `decimal(18,2)` |
| Measurements / quantities | `decimal(18,4)` |
| Percentage | `decimal(5,2)` |
| GUID / public identifier | `uniqueidentifier` |
| Binary | `varbinary(n)` / `varbinary(max)` |

Defaults: ``default: `SYSUTCDATETIME()` ``, ``default: `NEWID()` ``, `default: 1`, `default: 0`.
Avoid `text`, `ntext`, `image`, `float`, `real`, `datetime`, `smalldatetime`.
In `Note`s, mention filtered unique indexes when a unique column is nullable (SQL Server allows only one NULL in a plain unique index).

---

## 7. Normalization (target: 3NF)

- **1NF**: atomic values; no comma-separated lists, no repeating columns (`Phone1`, `Phone2`) → child table.
- **2NF**: with composite keys, every attribute depends on the whole key.
- **3NF**: no transitive dependencies; an attribute depending on another non-key attribute moves to its own entity.
- **N:M**: junction table with own `Id`, audit columns, and a composite `unique` index on the FK pair.
- Derived/calculable values (totals, ages, running stock) are not stored unless justified in a `Note` and listed under `## Observations`.

---

## 8. Modules and section flags (standard)

Every table belongs to exactly one **module** (a functional area: users, API, finance, a department, a feature, an integration…). Modules must be obvious when scanning the file.

### 8.1 Module banner (mandatory)
```dbml
// ============================================================
// [MODULE:01] IDENTITY & ACCESS
// Scope : Accounts, roles, permissions and authentication data.
// Tables: UserType, User, Role, UserRole
// ============================================================
```
- Tag format: `[MODULE:NN] TITLE IN UPPERCASE`, with `NN` a two-digit sequential number. The tag is greppable.
- Required lines: `Scope` (one sentence) and `Tables` (comma-separated, in file order).
- Order modules from foundational (identity, catalogs) to dependent (operations, finance, reporting).

### 8.2 Sub-section flags (optional, inside a module)
```dbml
// ---- [01.1] Catalogs ----
// ---- [01.2] Entities ----
// ---- [01.3] Junctions ----
```

### 8.3 Relationships block (at the end of each module)
```dbml
// ---- [01.R] Relationships ----
Ref: User.IdUserType > UserType.Id
```

### 8.4 TableGroup (mandatory, one per module)
```dbml
TableGroup IdentityAccess {
  UserType
  User
  Role
  UserRole
  Note: 'Module 01 - Identity & Access'
}
```
`TableGroup` name = module title in UpperCamelCase without `&`. This gives the visual grouping in dbdiagram.io.

### 8.5 Cross-module references
A FK pointing to another module's table is declared in the relationships block of the module that **owns the FK column**, with a trailing comment: `// -> [MODULE:01]`.

Audit FKs (`IdCreatedBy`, `IdModifiedBy`) to `User` are declared in each module's relationship block.

---

## 9. Relationships

- Syntax: `Ref: Table.IdColumn > ParentTable.Id` (`>` many-to-one, `-` one-to-one, `<>` many-to-many only for documentation; prefer junction tables).
- Every `Id*` column other than `Id` must have a `Ref`. Never use invalid operators.
- Never delete existing `Ref`s; update column names if they changed.

---

## 10. Documentation (mandatory)

- Every **table**: `Note: '...'` with purpose and business context.
- Every **column**: `note: '...'` with meaning, allowed values, constraints and policy (e.g. hashing algorithm, UTC, bootstrap).
- Use `indexes { }` for composite uniques and frequent lookups, with a `note`.
- Notes in English, one or two sentences.

---

## 11. Procedure

1. Read the input (concepts, fields or existing DBML).
2. Assign each table to a module (section 8).
3. Apply naming (section 2).
4. Detect categorical fields → lookups (section 5).
5. Normalize to 3NF (section 7).
6. Add mandatory columns and order them (sections 3–4).
7. Map types to SQL Server (section 6).
8. Declare `Ref`s and `TableGroup`s (sections 8–9).
9. Add notes (section 10).
10. Return the complete DBML; append `## Observations` only if needed.

## Final checklist
- [ ] Entities singular, UpperCamelCase, English
- [ ] PK = `Id`; FKs = `Id<Entity>`
- [ ] Timestamps end with `At` (`datetime2`); dates end with `Date` (`date`)
- [ ] `Description` (or `Name` only for persons/users/accounts)
- [ ] All 6 mandatory columns in every table
- [ ] `RecordStatus` is `tinyint`
- [ ] Column order: Id → descriptive → dates → RecordStatus → FKs
- [ ] Categorical fields extracted into lookup tables
- [ ] Every table inside a `[MODULE:NN]` banner and a `TableGroup`
- [ ] Every FK has a `Ref`
- [ ] Notes on every table and column
