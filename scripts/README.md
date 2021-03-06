# Scripts

Files in this folder can be used for multiple purposes such as VSCode compilation, build scripts, etc

- [Files](#files)
- [`apex_disable.sql`](#apex_disablesql)
- [`apex_export_app.sql`](#apex_export_appsql)
- [`helper.sh`](#helpersh)
  - [Environment Variables](#environment-variables)
  - [Functions](#functions)
    - [`export_apex_app`](#export_apex_app)
    - [`reset_release`](#reset_release)
    - [`list_all_files`](#list_all_files)
- [`project-config.sh`](#project-configsh)
- [`user-config.sh`](#user-configsh)

## Files

**You will need to configure both `project-config.sh` and `user-config.sh` upon first use**

File | Description
--- | ---
[`apex_export_app.sql`](#apex_export_appsql) | Exports an APEX application
[`helper.sh`](#helpersh) | Helper functions that all other scripts should call. Loads `config.sh`
`project-config.sh` | Project configuration
[`user-config.sh`](#user-configsh) | This file will be automatically generated when any bash script is run for the first time. It is self documenting.


## `apex_disable.sql`

This script will disable (sets the application's status to Unavailable). It's main purpose is to disable the application at the start of a release so users don't use it while the schema is being upgraded. By default this is called in [`../release/_release.sql`].

**This script performs a `commit` at the end.**

Parameter | Description
--- | ---
`1` | Comma delimited list of application IDs

Example:

```bash
echo exit | sqlcl martin/password123@localhost:32118/xe @apex_disable.sql 100,200
```



## `apex_export_app.sql`

This script requires [SQLcl](https://www.oracle.com/ca-en/database/technologies/appdev/sqlcl.html)

Parameter | Description
--- | ---
`1` | Application ID
`2` | *Optional* APEX Export options (ex: `-split`)


Examples:

```bash
# Export to f100.sql
echo exit | sqlcl martin/password123@localhost:32118/xe @apex_export_app.sql 100

# Export to Application 100 as split files
echo exit | sqlcl martin/password123@localhost:32118/xe @apex_export_app.sql 100 -split
```


## `helper.sh`

Documentation below references `project root folder`. This is the base folder that the git project is located on your computer. Ex: `/Users/martin/git/insum/starter-project-template/`

To load the helper functions run: `source scripts/helper.sh` (*assuming you are in the project's root folder*). This will load some environment variables and load/verify configurations for this release.

### Environment Variables

Name | Description
--- | ---
`SCRIPT_DIR` | Directory that the helper file is located in. Using the example above this will return: `/Users/martin/git/insum/starter-project-template/scripts`
`PROJECT_DIR` | Root directory of the git repo that is associated to this file. Ex: `/Users/martin/git/insum/starter-project-template/`


### Functions

#### `export_apex_app`

Exports APEX applications and also splits the export file. APEX exports will be stored in `<project_root>/apex` folder. The list of applications to export is defined in `scripts/project-config.sh` variable `APEX_APP_IDS`

**Parameters**
Position | Required | Description
--- | --- | ---
`$1` | Optional | Application version. If defined will search the exported application file for `%RELEASE_VERSION%` and replaced it with this variable. See the [`apex`](../apex/) documentation for more information.

**Example**

```bash
# No app version
export_apex_app

# App version
export_apex_app 1.2.3
```


#### `reset_release`

Resets the project's root release folder. Because resetting will erase everything in the `release/code` folder and reset `release/code/_run_code.sql` this function requires that an additional parameter is passed in to ensure that nothing is deleted by mistake.

**Parameters**
Position | Required | Description
--- | --- | ---
`$1` | Required | project root directory name. If this root folder is `/Users/martin/git/insum/starter-project-template/` then this parameter will be `starter-project-template`

**Example**

```bash
# Show what happens when no parameter is passed in
# Note the error message will show what call to make
reset_release 

Error:  confirmation directory missing or not matching. Run: reset_release starter-project-template

# Correct run

reset_release starter-project-template
```


#### `list_all_files`

It is very rare that you'd need to run this function on it's own as it's called as part of the [`build`](../build) process. This function will list all the files in a folder and output the results with `@../` prefix in a specified output file. This is useful when wanting to automatically compile all packages and views as part of the build.

**Parameters**
Position | Required | Description
--- | --- | ---
`$1` | Required | Folder that you want to list files from. **Note:** this folder is the folder name **relative** to the project's root folder. I.e. for `views` you would specify `views` and **not** `/Users/martin/git/insum/starter-project-template/views`
`$2` | Required | File to store results in
`$3` | Optional | Comma delimited list of file extensions to search for. Default: `sql`. Note the order of the list matters. For example if `pks,pkb` all the `pks` (spec) files will be listed first then the `pkb` (body) files will be listed second.

**Example**
```bash
# Generate all the views
list_all_files views release/all_views.sql sql

# Generate all the packages
# Note pks is before pkb so that the specs get listed before the body
list_all_files packages release/all_packages.sql pks,pkb
```

## `project-config.sh`
This file contains information about your project (such as schema name, APEX applications, etc.). It is common for all developers and changes are saved in git. **Do not** put any sensitive information in this file (`user-config.sh` is for sensitive information).

## `user-config.sh`
The first time you run any bash script an error will be displayed and a new file (`scripts/user-config.sh`) will be created. `user-config.sh` is self documented and requires some configuration before the build will work.

`user-config.sh` is in the `.gitignore` file so you can store more sensitive information without it being checked in.