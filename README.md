# SqlDeployer
SQL Migration Management and Deployment Tool



## Command Line Switches

Usage:
  SqlDeployer [options]

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
  --accesstoken <accesstoken>                            Access token to be used for logging in to SQL Server / Azure
                                                         SQL Database.

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
  
  ## Scripts Exrcution Order
  
  Scripts within directories are being exrcuted in alphabetical order.
  
  Good practice is to name them with numbered prefix, ex. 00001-ScriptName.sql

  ## Directory roles and execution order

  | Order                       | Directory                   | Purpose                                                                                      | Track script changes                                                       |
|-----------------------------|-----------------------------|----------------------------------------------------------------------------------------------|----------------------------------------------------------------------------|
| BeforeMigration             | PrepareForDeployment        | SQL Server Wide PReparation Scripts (ex. Replication Pausing, Database Creation)             | False                                                                      |
| RunAfterCreateDatabase      | RunAfterCreateDatabase      | Scripts run after database is created (ex. Database Options)                                 | False                                                                      |
| AlterDatabase               | AlterDatabase               | Scripts altering database itself (database wide, not schema objects!). Ex. Isolation Levels. | False                                                                      |
| RunBeforeUp                 | RunBeforeMigration          | Scripts tpo be run before migration (ex. user disconnects, RESTRICTED_USER, SINGLE_USER)     | False                                                                      |
| Up                          | Migrate                     | Default directory for the scripts (tables, schema/object alters, etc.)                       | False                                                                      |
| RunFirstAfterUp             | RunFirstAfterMigration      | Scripts performed after the migration (ex. data manipulation)                                | False                                                                      |
| Functions                   | Functions                   | Scripts creating/altering Functions                                                          | False                                                                      |
| View                        | Views                       | Scripts creating/altering Views                                                              | False                                                                      |
| StoredProcedures            | Sprocs                      | Scripts creating/altering Stored Procedures                                                  | False                                                                      |
| Indexes                     | Indexes                     | Scripts creating/altering Indexes                                                            | False                                                                      |
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
