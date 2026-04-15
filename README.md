# SqlDeployer
SQL Migration Management and Deployment Tool

[![NuGet](https://img.shields.io/nuget/v/SqlDeployer.svg)](https://www.nuget.org/packages/SqlDeployer/)
[![License](https://img.shields.io/github/license/AnubisWorks/SqlDeployer)](LICENSE)

## Features

✅ **Multi-Database Support**
- SQL Server
- PostgreSQL
- MariaDB / MySQL
- Oracle
- SQLite

✅ **Migration Management**
- Script-based migrations with numbered ordering
- Environment-specific scripts
- Transaction support
- Token replacement
- Pre/Post deployment scripts

✅ **Reverse Engineering** (New in v10.3.0)
- Extract database schema to SQL scripts
- Support for all major database systems
- Generate numbered migration baselines
- Version control ready output

✅ **Enterprise Features** (New in v10.4.0)
- Pre-migration backup (SQLite auto-copy, SQL Server BACKUP, advisory for PG/MariaDB/Oracle)
- Verify scripts (`verify/` folder convention — auto-run after one-time migrations)
- Shell script hooks (`before-migrate`, `after-migrate`, `before/after-each-script`, `on-error`)
- 50 SQL lint rules (backward-incompatible changes, locking, security, schema design)
- Migration simulation (`sqldeployer simulate` — test on shadow database copy)
- Expand-contract pattern (`--strategy expand-contract`, `--finalize`)
- Policy engine (`.sqldeployer-policy.json` — per-environment enforcement)
- Signed audit trail (HMAC-SHA256, tamper detection, export JSON/CSV)
- Approval gates (four-eyes principle — `sqldeployer approve`)
- Encrypted connection vault (`sqldeployer vault set/get/list/delete`)
- Impact analysis (`sqldeployer analyze` — risk scoring, dependency graph)
- Multi-tenant migration (`--tenant-list`, per-tenant tracking, `--retry-failed`)
- Performance profiling (`--profile` — per-script timing, auto-warnings)
- OpenTelemetry instrumentation (`--otel-endpoint`)
- Resilience (retry with exponential backoff, circuit breaker)
- Plugin system (`ISqlDeployerPlugin` interface, NuGet-based discovery)

✅ **ORM Integration** (New in v10.4.0)
- SqlFactory (AnubisWorks) — reflection-based schema reader
- EF Core — DbContext/DbSet discovery + DataAnnotations
- Declarative schema apply (`sqldeployer schema apply` — desired state diff)
- Computed rollback (auto-generated Down/ scripts)

✅ **Containerized Deployments**
- Docker support
- Kubernetes ready
- CI/CD integration examples

## Quick Start

### Installation

```bash
# Install as global .NET tool
dotnet tool install -g SqlDeployer

# Verify installation
sqldeployer --version
```

### Run Migrations

```bash
sqldeployer migrate \
  --connectionstring "Server=localhost;Database=MyDb;..." \
  --databasetype SQLServer \
  --sqlfilesdirectory ./migrations
```

### Reverse Engineer Database

```bash
sqldeployer reverse \
  --connectionstring "Server=localhost;Database=MyDb;..." \
  --databasetype SQLServer \
  --outputdirectory ./schema-baseline
```

## Documentation

- [Getting Started](docs/GettingStarted.md)
- [Reverse Engineering Guide](docs/ReverseEngineering.md)
- [Configuration Options](docs/ConfigurationOptions/)
- [Script Types](docs/ScriptTypes/)
- [Docker Setup](docs/DOCKER_SETUP.md)
- [Build and Deployment](docs/BUILD_AND_DEPLOYMENT.md)
- [Test Configuration](docs/TEST_CONFIGURATION.md) - Fast vs Full Tests
- [Token Replacement](docs/TokenReplacement.md)



## Command Line Switches

Usage:
  dotnet SqlDeployer [options]

Options:
  -c, -cs, --connectionstring, --connstring              You now provide an entire connection string.

  -f, --files, --sqlfilesdirectory <sqlfilesdirectory>   The directory where your SQL scripts are [default: .]

  -o, --output, --outputPath <outputPath>                This is where everything related to the migration is stored.
                                                         This includes any backups, all items that ran, permission
                                                         dumps, logs, etc. [default:
                                                         C:\Users\{username}\AppData\Local\sqldep]

  --folders <folders>                                    Folder configuration.

                                                         You can also specify a file name that has the same contents as
                                                         you would supply on the command line.

                                                         Example:

                                                           --folders myfolderconfig.txt

                                                         ,and put the following content in `myfolderconfig.txt`(either
                                                         semicolon or newline separated)

                                                         up=Promote
                                                         views=ViewsCustom

                                                         this will keep the default folder configuration, but change
                                                         the names of the folders you supply.

  -a, -acs, -csa, --adminconnectionstring,               The connection string for connecting to master, if you want to
  --adminconnstring <adminconnectionstring>              create the database.  Defaults to the same as --connstring.
  
  --accesstoken <accesstoken>                            (Optional) Access token for SQL Server / Azure SQL. When set,
                                                         it will be assigned to every SqlConnection; if not provided,
                                                         the application functions as before without token.

  -ct, --commandtimeout <commandtimeout>                 This is the timeout when commands are run. This is not for
                                                         admin commands or restore. [default: 60]
  
  -cta, --admincommandtimeout <admincommandtimeout>      This is the timeout when administration commands are run
                                                         (except for restore, which has its own). [default: 300]
  
  --databasetype, --dbt, --dt                            Points SqlDeployer what type of database it is running on.
  <mariadb|oracle|postgresql|sqlite|sqlserver>           [default: sqlserver]
  
  -t, --transaction, --trx                               Run the migration in a transaction

  --env, --environment <environment>                     Environment Name - This allows SqlDeployer to be environment
                                                         aware and only run scripts that are in a particular
                                                         environment based on the name of the script.
                                                         'something.ENV.LOCAL.sql' would only be run if --env=LOCAL was
                                                         set.
  
  --sc, --schema, --schemaname <schemaname>              The schema to use for the migration tables [default: sqldep]
  
  -ni, --noninteractive, --silent                        Silent - instructs SqlDeployer not to ask for any input when
                                                         it runs.
  
  --version <version>                                    Database Version - specify the version of the current
                                                         migration directly on the command line.
  
  --drop                                                 Drop - This instructs SqlDeployer to remove the target
                                                         database.
  
  --create, --createdatabase                             Create - This instructs SqlDeployer to create the target
                                                         database if it does not exist.  Defaults to true. [default:
                                                         True]
  
  --disabletokenreplacement, --disabletokens             Tokens - This instructs SqlDeployer to not perform token
                                                         replacement ({{somename}}). Defaults to false.
  
  --usertokens, --ut <usertokens>                        User Tokens - Allows SqlDeployer to perform token replacement
                                                         on custom tokens ({{my_token}}). Set as a key=value pair, eg
                                                         '--ut=my_token=myvalue'. Can be specified multiple times.
  
  --baseline                                             Baseline - This instructs SqlDeployer to mark the scripts as
                                                         run, but not to actually run anything against the database.
                                                         Use this option if you already have scripts that have been run
                                                         through other means (and BEFORE you start the new ones).
  
  --forceanytimescripts, --runallanytimescripts          RunAllAnyTimeScripts - This instructs SqlDeployer to run any
                                                         time scripts every time it is run even if they haven't
                                                         changed. Defaults to false.
  
  --dryrun                                               DryRun - This instructs SqlDeployer to log what would have
                                                         run, but not to actually run anything against the database.
                                                         Use this option if you are trying to figure out what
                                                         SqlDeployer is going to do.
  
  --restore <restore>                                    Restore - This instructs SqlDeployer where to get the backed
                                                         up database file. Defaults to NULL.
  
  -v, --verbosity                                        Verbosity level (as defined here:
  <Critical|Debug|Error|Information|None|Trace|Warning>  https://docs.microsoft.com/dotnet/api/Microsoft.Extensions.Logging.LogLevel)
  
  -?, -h, --help                                         Show help and usage information
  
  ## Scripts Execution Order
  
  Scripts within directories are being executed in alphabetical order.
  
  Good practice is to name them with numbered prefix, ex. 00001-ScriptName.sql

  ## Directory roles and execution order

| Order                       | Directory                   | Purpose                                                                                      | Track script changes                                                       |
|-----------------------------|-----------------------------|----------------------------------------------------------------------------------------------|----------------------------------------------------------------------------|
| BeforeMigration             | PrepareForDeployment        | SQL Server Wide PReparation Scripts (ex. Replication Pausing, Database Creation)             | True                                                                       |
| RunAfterCreateDatabase      | RunAfterCreateDatabase      | Scripts run after database is created (ex. Database Options)                                 | False                                                                      |
| AlterDatabase               | AlterDatabase               | Scripts altering database itself (database wide, not schema objects!). Ex. Isolation Levels. | True                                                                       |
| RunBeforeUp                 | RunBeforeMigration          | Scripts to be run before migration (ex. user disconnects, RESTRICTED_USER, SINGLE_USER)     | False                                                                      |
| Up                          | Migrate                     | Default directory for the scripts (tables, schema/object alters, etc.)                       | False                                                                      |
| RunFirstAfterUp             | RunFirstAfterMigration      | Scripts performed after the migration (ex. data manipulation)                                | True                                                                       |
| Functions                   | Functions                   | Scripts creating/altering Functions                                                          | True                                                                       |
| View                        | Views                       | Scripts creating/altering Views                                                              | True                                                                       |
| StoredProcedures            | Sprocs                      | Scripts creating/altering Stored Procedures                                                  | True                                                                       |
| Indexes                     | Indexes                     | Scripts creating/altering Indexes                                                            | True                                                                       |
| Triggers                    | Triggers                    | Scripts creating/altering Triggers                                                           | False                                                                      |
| Permissions                 | Permissions                 | Scripts creating/altering Permissions                                                        | False                                                                      |
| RunAfterOtherAnyTimeScripts | RunAfterOtherAnyTimeScripts | Other scripts, that should be run                                                            | True (Existing scripts may be changed and shall be executed with tracking) |
| AfterMigration              | AfterMigration              | Scripts run after whole migration (ex. database set MULTI_USER)                              | False                                                                      |

## Example FolderConfig file

```
BeforeMigration=PrepareForDeployment
AlterDatabase=AlterDatabase
RunAfterCreateDatabase=RunAfterCreateDatabase
RunBeforeUp=RunBeforeMigration
Up=Migrate
RunFirstAfterUp=RunFirstAfterMigration
Functions=Functions
Views=Views
Sprocs=StoredProcedures
Triggers=Triggers
Indexes=Indexes
RunAfterOtherAnyTimeScripts=RunAfterOtherAnyTimeScripts
Permissions=Permissions
AfterMigration=RunAfterMigration
```
    
## Everytime Scripts
    
If you name your file with .EVERYTIME., SqlDeployer will run that file everytime, regardless of changes. For example, if you name a file Sth.EVERYTIME.sql or EVERYTIME.sth.sql, SqlDeployer will run that file on each migration.
    
## Environment Scripts
  
Environment Name - This allows SqlDeployer to be environment aware and only run scripts that are in a particular environment based on the name of the script. 'sth.ENV.LOCAL.sql' would only be run if --env=LOCAL was set.

## Enterprise Commands

```bash
# Check migration status
sqldeployer status -c "..." --dt SQLServer

# Validate scripts (hash integrity, tokens, orphaned records)
sqldeployer validate -c "..." --dt SQLServer -f ./migrations

# Lint SQL scripts (50 rules)
sqldeployer lint -f ./migrations

# Compare two database schemas
sqldeployer diff -c "Server=A;..." --target-connectionstring "Server=B;..." --dt SQLServer

# Detect schema drift
sqldeployer drift -c "..." --dt SQLServer --snapshot ./baseline.json

# Impact analysis before migration
sqldeployer analyze -c "..." --dt SQLServer -f ./migrations

# Simulate migration on shadow database
sqldeployer simulate -c "..." --dt SQLite -f ./migrations

# Manage encrypted connection vault
sqldeployer vault set production -c "Server=prod;..."
sqldeployer vault get production

# Audit trail
sqldeployer audit --export csv --export-path audit.csv

# Schema from ORM
sqldeployer schema from-sqlfactory --assembly ./MyApp.dll -o schema.json
sqldeployer schema from-efcore --assembly ./MyApp.dll -o schema.json
sqldeployer schema apply --schema-file schema.json -c "..."
```

## Releases

SqlDeployer releases are available on GitHub at [michalagata/SqlDeployer](https://github.com/michalagata/SqlDeployer).

Each release includes:
- Platform-specific ZIP archives (Linux, macOS, Windows)
- NuGet dotnet tool package (`.nupkg`)
- `version.txt` file containing the release version
- `README.md` file with documentation

### Installation Options

```bash
# Option 1: Install as global .NET tool (recommended)
dotnet tool install -g SqlDeployer

# Option 2: Download from GitHub Releases
# https://github.com/michalagata/SqlDeployer/releases
# Select: SqlDeployer.Linux.zip / SqlDeployer.macOS.zip / SqlDeployer.Windows.zip
```
