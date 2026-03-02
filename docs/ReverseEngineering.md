# Reverse Engineering - Quick Start

## Overview

SqlDeployer now supports reverse engineering database schemas to SQL migration scripts. This feature allows you to:
- Generate baseline SQL scripts from existing databases
- Document database structure in version control
- Create migration starting points for new projects
- Compare database schemas across environments

## Supported Databases

- **SQL Server** ✅ (Fully implemented)
- PostgreSQL 🚧 (Coming soon)
## Supported Database Systems

- **SQL Server** ✅ (Fully supported)
- **PostgreSQL** ✅ (Fully supported)
- **MariaDB / MySQL** ✅ (Fully supported)
- **Oracle** ✅ (Fully supported)
- **SQLite** ✅ (Fully supported - limited features)

### Dialect-Specific Features

| Feature | SQL Server | PostgreSQL | MariaDB/MySQL | Oracle | SQLite |
|---------|-----------|-----------|--------------|--------|--------|
| Tables | ✅ | ✅ | ✅ | ✅ | ✅ |
| Views | ✅ | ✅ | ✅ | ✅ | ✅ |
| Procedures | ✅ | ✅ | ✅ | ✅ | ❌ |
| Functions | ✅ | ✅ | ✅ | ✅ | ❌ |
| Triggers | ✅ | ✅ | ✅ | ✅ | ✅ |
| Indexes | ✅ | ✅ | ✅ | ✅ | ✅ |
| Foreign Keys | ✅ | ✅ | ✅ | ✅ | ⚠️ (inline only) |
| Schemas | ✅ | ✅ | ✅ | ✅ | ❌ (single schema) |

## Quick Start

### Basic Usage

```bash
# Reverse engineer entire database
sqldeployer reverse \
  --connectionstring "Server=localhost;Database=MyDb;Integrated Security=true" \
  --databasetype SQLServer \
  --outputdirectory ./sql-baseline

# Filter by schema
sqldeployer reverse \
  -c "Server=localhost;Database=MyDb;Integrated Security=true" \
  -d SQLServer \
  -o ./sql-baseline \
  --schema dbo

# Select specific object types
sqldeployer reverse \
  -c "Server=localhost;Database=MyDb;Integrated Security=true" \
  -d SQLServer \
  -o ./sql-baseline \
  --objects tables,views,procedures

# Organize in flat structure (no subfolders)
sqldeployer reverse \
  -c "Server=localhost;Database=MyDb;Integrated Security=true" \
  -d SQLServer \
  -o ./sql-baseline \
  --folders false
```

### Command Options

| Option | Alias | Required | Default | Description |
|--------|-------|----------|---------|-------------|
| `--connectionstring` | `-c` | ✅ | - | Database connection string |
| `--databasetype` | `-d` | ✅ | - | Database type (SQLServer, PostgreSQL, etc.) |
| `--outputdirectory` | `-o` | ❌ | `./sql-baseline` | Output directory for SQL scripts |
| `--schema` | `-s` | ❌ | (all) | Filter by schema name |
| `--objects` | `-obj` | ❌ | (all) | Object types: `tables`, `views`, `procedures`, `functions`, `indexes`, `foreignkeys`, `triggers` |
| `--folders` | `-f` | ❌ | `true` | Organize scripts into subfolders by object type |
| `--includedata` | - | ❌ | `false` | Include INSERT statements for data (coming soon) |

## Output Structure

### With Folders (default)

```
sql-baseline/
├── tables/
│   ├── 00001__dbo_Users.sql
│   ├── 00002__dbo_Orders.sql
│   └── 00003__dbo_OrderItems.sql
├── views/
│   ├── 00004__dbo_vw_UserOrders.sql
│   └── 00005__dbo_vw_ActiveUsers.sql
├── procedures/
│   ├── 00006__dbo_sp_GetUserOrders.sql
│   └── 00007__dbo_sp_CreateOrder.sql
├── functions/
│   └── 00008__dbo_fn_CalculateTotal.sql
├── indexes/
│   ├── 00009__dbo_Users_IX_Email.sql
│   └── 00010__dbo_Orders_IX_UserId.sql
├── foreignkeys/
│   ├── 00011__dbo_Orders_FK_UserId.sql
│   └── 00012__dbo_OrderItems_FK_OrderId.sql
└── triggers/
    └── 00013__dbo_Users_tr_AuditLog.sql
```

### Without Folders

```
sql-baseline/
├── 00001__tables_dbo_Users.sql
├── 00002__tables_dbo_Orders.sql
├── 00003__views_dbo_vw_UserOrders.sql
├── 00004__procedures_dbo_sp_GetUserOrders.sql
└── ...
```

## SQL Server Details

### Supported Objects

✅ **Tables**
- Complete column definitions (data types, nullability, defaults, computed columns)
- Primary keys (inline in CREATE TABLE)
- Identities, collations, precision/scale

✅ **Views**
- Original DDL extracted from system catalogs
- Dependencies preserved

✅ **Stored Procedures**
- Original DDL with parameters and body
- CLR procedures (original definition)

✅ **Functions**
- Scalar, inline table-valued, multi-statement table-valued
- Original DDL preserved

✅ **Indexes**
- Clustered/non-clustered
- Unique constraints
- Included columns
- Filter predicates (coming soon)

✅ **Foreign Keys**
- Multi-column foreign keys
- ON DELETE/UPDATE actions
- Cross-schema references

✅ **Triggers**
- AFTER and INSTEAD OF triggers
- Original DDL preserved

### System Views Used

SqlDeployer uses official SQL Server system catalog views:
- `sys.tables`, `sys.columns`, `sys.types`
- `sys.views`, `sys.procedures`, `sys.sql_modules`
- `sys.indexes`, `sys.index_columns`
- `sys.foreign_keys`, `sys.foreign_key_columns`
- `sys.triggers`, `sys.schemas`

## Examples

### Example 1: SQL Server - Generate Baseline for New Project

```bash
# Extract schema from production database
sqldeployer reverse \
  -c "Server=prod-server;Database=MyApp;User=sa;Password=****" \
  -d SQLServer \
  -o ./migrations/baseline

# Result: Numbered SQL scripts ready for version control
```

### Example 2: PostgreSQL - Extract Public Schema

```bash
sqldeployer reverse \
  -c "Host=localhost;Database=myapp;Username=postgres;Password=****" \
  -d PostgreSQL \
  -o ./schema-docs \
  --schema public
```

### Example 3: MariaDB/MySQL - Core Tables Only

```bash
sqldeployer reverse \
  -c "Server=localhost;Database=myapp;User=root;Password=****" \
  -d MariaDB \
  -o ./schema-docs \
  --objects tables,indexes,foreignkeys
```

### Example 4: Oracle - Extract Specific Schema

```bash
sqldeployer reverse \
  -c "Data Source=localhost:1521/ORCL;User Id=system;Password=****" \
  -d Oracle \
  -o ./schema-docs \
  --schema MYAPP
```

### Example 5: SQLite - Full Database Export

```bash
sqldeployer reverse \
  -c "Data Source=/path/to/myapp.db" \
  -d SQLite \
  -o ./schema-docs
```

### Example 6: Multi-Database CI/CD Schema Documentation

```yaml
# .github/workflows/schema-doc.yml
name: Document Schema
on:
  push:
    branches: [main]
jobs:
  document:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        database: [SQLServer, PostgreSQL, MariaDB]
    steps:
      - uses: actions/checkout@v2
      - name: Install SqlDeployer
        run: dotnet tool install -g SqlDeployer.Cli
      - name: Generate Schema
        run: |
          sqldeployer reverse \
            -c "${{ secrets[format('DB_CONNECTION_{0}', matrix.database)] }}" \
            -d ${{ matrix.database }} \
            -o ./docs/schema/${{ matrix.database }}
      - name: Commit Changes
        run: |
          git add docs/schema
          git commit -m "Update schema documentation"
          git push
```

## Integration with Migration Workflow

Generated scripts can be used as baseline for SqlDeployer migrations:

```bash
# 1. Generate baseline
sqldeployer reverse -c "..." -d SQLServer -o ./db/baseline

# 2. Copy to up folder
cp -r ./db/baseline/* ./db/migrations/up/

# 3. Run migrations on clean database
sqldeployer migrate -c "..." -d SQLServer --sqlfilesdirectory ./db/migrations
```

## Troubleshooting

### Connection Issues

```bash
# Test connection first
sqldeployer reverse -c "Server=localhost;Database=master;..." -d SQLServer -o /tmp/test
```

### Permission Requirements

SQL Server introspection requires:
- `VIEW DEFINITION` permission on database
- `SELECT` on system views (`sys.*`)
- Typically `db_datareader` role is sufficient

### Large Databases

For databases with 1000+ objects, consider:
- Filter by schema: `--schema dbo`
- Extract object types separately
- Use flat structure for better performance: `--folders false`

## Roadmap

- ✅ SQL Server support (v10.3.0)
- ✅ PostgreSQL support (v10.3.0)
- ✅ MariaDB/MySQL support (v10.3.0)
- ✅ Oracle support (v10.3.0)
- ✅ SQLite support (v10.3.0)
- 🚧 Data export (--includedata flag implementation)
- 🚧 EF Core model reverse engineering
- 🚧 Oracle support (Q2 2026)
- 🚧 SQLite support (Q2 2026)
- 🚧 Include data (INSERT statements)
- 🚧 EF Core model → SQL scripts
- 🚧 Differential schema comparison
- 🚧 Custom script templates

## See Also

- [Main Documentation](../index.md)
- [Getting Started](GettingStarted.md)
- [Configuration Options](ConfigurationOptions/)
- [Build & Deployment](BUILD_AND_DEPLOYMENT.md)
