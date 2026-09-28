# OdooDev

Development environment for Odoo using [devenv](https://devenv.sh/).

The goal of this repository is to provide a reproducible local development
environment for Odoo projects.

## Prerequisites

Install:

- Git
- Nix
- devenv

## Setup

### 1. Clone the template

Clone the Odoo development template and rename it for the new project.

```bash
git clone git@github.com:samniyonkuru/000-odooDev.git
mv 000-odooDev 001-project
cd 001-project
```

Replace `001-project` with the name of the new project.

### 2. Create a new Git repository

Remove the Git history of the template and initialize a new repository for the project.

```bash
rm -rf .git
git init
git branch -M main
```

The project is now independent from `000-odooDev`.

### 3. Clone Odoo

The Odoo source code is kept locally and is not tracked by the project repository.

#### Odoo 19

```bash
git clone --depth 1 --branch 19.0 https://github.com/odoo/odoo.git odoo
```

#### Odoo 20

```bash
git clone --depth 1 --branch 20.0 https://github.com/odoo/odoo.git odoo
```

Choose the Odoo version required by the project.

The resulting structure should look like:

```text
001-project/
├── custom-addons/
├── odoo/
├── devenv.nix
├── devenv.yaml
├── devenv.lock
├── .gitignore
└── README.md
```

The `odoo/` directory is ignored by Git.

### 4. Start the development environment

The devenv configuration is already included in the template.

```bash
devenv shell
```

There is no need to run:

```bash
devenv init
```

The development environment provides Python, a Python virtual environment,
PostgreSQL and the other dependencies required by the project.

### 5. Install Odoo Python requirements

Install the Python dependencies required by Odoo inside the devenv virtual environment.

```bash
pip install -r odoo/requirements.txt
```

You can verify that the dependencies are available with:

```bash
python -c "import babel; print(babel.__version__)"
```

### 6. Start PostgreSQL

Start the devenv services in the background:

```bash
devenv processes up -d
```

Check that PostgreSQL is running:

```bash
pg_isready
```

The result should contain:

```text
accepting connections
```

You can also check the existing databases:

```bash
psql -l
```

### 7. Remove the automatically created user database

PostgreSQL may automatically create a database with the same name as the
current Linux user.

For example:

```text
postgres
sam
template0
template1
```

The `sam` database in this example is a PostgreSQL database created for the
local user. It is not an initialized Odoo database.

Remove the database corresponding to the current Linux user:

```bash
dropdb "$USER"
```

Check the databases again:

```bash
psql -l
```

For a clean environment, the PostgreSQL system databases should remain:

```text
postgres
template0
template1
```

### 8. Generate the Odoo configuration

Generate the local Odoo configuration file:

```bash
python odoo/odoo-bin \
  --config=./odoo.conf \
  --save
```

Odoo will start after saving the configuration.

Stop it with:

```text
Ctrl+C
```

The generated configuration contains the settings for the current development
environment, including the PostgreSQL connection.

The file is created at:

```text
001-project/odoo.conf
```

The `odoo.conf` file is local to the project and should not be committed.

Make sure `.gitignore` contains:

```gitignore
odoo.conf
```

### 9. Start Odoo

Start Odoo using the generated configuration:

```bash
python odoo/odoo-bin -c ./odoo.conf
```

Odoo should report:

```text
Odoo version 19.0
HTTP service (werkzeug) running on ...:8069
```

Odoo is now available at:

```text
http://localhost:8069
```

### 10. Create the Odoo database

Open the Odoo Database Manager:

```text
http://localhost:8069/web/database/manager
```

Create the database directly from the Odoo interface.

For a clean installation:

- Choose a database name
- Define the administrator email
- Define the administrator password
- Select the language
- Select the country
- Disable demo data

If the default generated configuration is used, the master password is:

```text
admin
```

This creates a fresh Odoo installation without pre-populating an Odoo database
from the command line.

## Daily workflow

The complete setup only needs to be performed once.

For normal development, enter the project:

```bash
cd ~/odoo/001-project
```

Enter the devenv environment:

```bash
devenv shell
```

Start PostgreSQL in the background:

```bash
devenv processes up -d
```

Optionally verify PostgreSQL:

```bash
pg_isready
```

Start Odoo:

```bash
python odoo/odoo-bin -c ./odoo.conf
```

Open Odoo:

```text
http://localhost:8069
```

When development is finished, stop Odoo with:

```text
Ctrl+C
```

Then stop the devenv services:

```bash
devenv processes down
```

## Project structure

```text
001-project/
├── custom-addons/
│   └── estate/
│       ├── __init__.py
│       ├── __manifest__.py
│       ├── models/
│       ├── security/
│       └── views/
│
├── odoo/              # Odoo source code - ignored by Git
├── odoo.conf          # Local configuration - ignored by Git
├── devenv.nix         # devenv configuration
├── devenv.yaml
├── devenv.lock
├── .gitignore
└── README.md
```

## Git

Only the project-specific environment and custom modules are tracked.

The Odoo source code and local configuration are excluded from the project
repository.

Example `.gitignore`:

```gitignore
# Odoo source
odoo/

# Odoo local configuration
odoo.conf

# devenv generated files
.devenv/
.direnv/

# Python
__pycache__/
*.pyc

# Logs
*.log
```

### Initial commit

Create the first commit:

```bash
git add .
git commit -m "Initial Odoo development environment"
```

Create a repository for the new project on GitHub and add it as the remote:

```bash
git remote add origin git@github.com:samniyonkuru/001-project.git
git push -u origin main
```

## Git workflow

For daily development:

```bash
git pull
git add .
git commit -m "Description of changes"
git push
```

## Architecture

```text
000-odooDev
      │
      │ clone
      ▼
001-project
      │
      ├── devenv
      │     ├── Python
      │     ├── Python virtual environment
      │     └── PostgreSQL
      │
      ├── Odoo source
      │
      ├── odoo.conf
      │
      └── custom-addons
            └── project-specific modules
```

`000-odooDev` acts as the development template.

Each new Odoo project gets its own:

- Git repository
- devenv environment
- PostgreSQL environment
- Odoo configuration
- Odoo database
- custom addons

The Odoo source code itself is cloned separately and is not committed to the
project repository.

## New project workflow summary

```text
Clone OdooDev template
        │
        ▼
Create new Git repository
        │
        ▼
Clone Odoo source
        │
        ▼
devenv shell
        │
        ▼
Install Odoo requirements
        │
        ▼
devenv processes up -d
        │
        ▼
dropdb "$USER"
        │
        ▼
Generate odoo.conf
        │
        ▼
Start Odoo
        │
        ▼
Open /web/database/manager
        │
        ▼
Create clean Odoo database
        │
        ▼
Start development
```
