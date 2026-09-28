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

Clone the Odoo development template directly into a directory named after the
new project.

```bash
git clone git@github.com:samniyonkuru/000-odooDev.git <project-name>
cd <project-name>
```

Example:

```bash
git clone git@github.com:samniyonkuru/000-odooDev.git 001-estate
cd 001-estate
```

Git clones the `000-odooDev` template directly into the `001-estate` directory.

The resulting directory is:

```text
001-estate/
```

### 2. Create a new Git repository

The cloned project still contains the Git history and remote of the template.

Remove them:

```bash
rm -rf .git
```

Initialize a new Git repository for the project:

```bash
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
001-estate/
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

The development environment provides:

- Python
- Python virtual environment
- PostgreSQL
- Project dependencies

### 5. Install Odoo Python requirements

Install the Python dependencies required by Odoo inside the devenv virtual
environment:

```bash
pip install -r odoo/requirements.txt
```

You can verify that the dependencies are available with:

```bash
python -c "import babel; print(babel.__version__)"
```

> If `devenv.nix` is configured to automatically install
> `odoo/requirements.txt`, this step is performed automatically when entering
> the devenv environment.

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

### 7. Remove the automatically created user database

PostgreSQL may automatically create a database with the same name as the
current Linux user.

Check the existing databases:

```bash
psql -l
```

Example:

```text
postgres
sam
template0
template1
```

In this example, `sam` is a PostgreSQL database created for the local user.
It is not an initialized Odoo database.

Remove the automatically created user database:

```bash
dropdb "$USER"
```

Check the databases again using the `postgres` database:

```bash
psql -d postgres -l
```

For a clean environment, the PostgreSQL system databases should remain:

```text
postgres
template0
template1
```

### 8. Generate the Odoo configuration

Generate the local Odoo configuration:

```bash
python odoo/odoo-bin \
  --config=./odoo.conf \
  --save
```

Odoo starts after saving the configuration.

Once the configuration has been generated, stop Odoo with:

```text
Ctrl+C
```

The generated file is:

```text
odoo.conf
```

It contains the settings for the current development environment, including
the PostgreSQL connection.

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

Odoo is available at:

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

The database is now initialized directly by Odoo.

---

## Daily workflow

The complete setup only needs to be performed once.

Enter the project:

```bash
cd ~/odoo/<project-name>
```

Example:

```bash
cd ~/odoo/001-estate
```

Enter the development environment:

```bash
devenv shell
```

Start PostgreSQL:

```bash
devenv processes up -d
```

Optionally check PostgreSQL:

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

### Stop the environment

Stop Odoo with:

```text
Ctrl+C
```

Then stop the devenv services:

```bash
devenv processes down
```

---

## Project structure

```text
<project-name>/
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

---

## Git

Only the project-specific environment and custom modules are tracked.

The Odoo source code and local configuration are excluded.

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

Create a GitHub repository for the new project and add it as the remote.

Example:

```bash
git remote add origin git@github.com:samniyonkuru/001-estate.git
git push -u origin main
```

---

## Git workflow

For daily development:

```bash
git pull
git add .
git commit -m "Description of changes"
git push
```

---

## Architecture

```text
000-odooDev
      │
      │ git clone ... <project-name>
      ▼
<project-name>
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

---

## New project workflow

```text
git clone 000-odooDev <project-name>
        │
        ▼
Remove template Git history
        │
        ▼
Initialize new Git repository
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
Start PostgreSQL
        │
        ▼
Remove automatic user database
        │
        ▼
Generate odoo.conf
        │
        ▼
Start Odoo
        │
        ▼
Create database from Odoo
        │
        ▼
Start development
```
