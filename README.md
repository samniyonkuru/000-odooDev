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

The development environment provides Python, the Python virtual environment,
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

> If `devenv.nix` is configured to automatically install
> `odoo/requirements.txt`, this step is performed automatically when entering
> the devenv environment.

### 6. Generate the Odoo configuration

Generate a local Odoo configuration file:

```bash
python odoo/odoo-bin \
  --config=./odoo.conf \
  --save
```

This creates:

```text
001-project/
└── odoo.conf
```

The configuration contains the settings for the current local environment,
including the PostgreSQL connection.

The `odoo.conf` file is local to the project and should be ignored by Git.

Make sure `.gitignore` contains:

```gitignore
odoo.conf
```

### 7. Start PostgreSQL

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

### 8. Start Odoo

Start the Odoo server using the local configuration:

```bash
python odoo/odoo-bin -c ./odoo.conf
```

Odoo is available at:

```text
http://localhost:8069
```

### 9. Create the Odoo database

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

If the default Odoo configuration is used, the master password is:

```text
admin
```

This creates a fresh Odoo installation without pre-populating a development
database from the command line.

## Daily workflow

Once the project has been configured, the complete setup does not need to be
repeated.

Enter the project:

```bash
cd ~/odoo/001-project
```

Enter the development environment:

```bash
devenv shell
```

Start PostgreSQL in the background:

```bash
devenv processes up -d
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

```bash
git add .
git commit -m "Initial Odoo development environment"
```

Add the project's GitHub repository:

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
