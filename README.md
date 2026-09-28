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
mv 000-odooDev 000-template
cd 000-template
```

### 2. Create a new Git repository

Remove the Git history of the template and initialize a new repository for the project.

```bash
rm -rf .git
git init
git branch -M main
```

The project is now independent from `000-odooDev`.

### 3. Start the development environment

The devenv configuration is already included in the template.

```bash
devenv shell
```

There is no need to run `devenv init`.

### 4. Clone Odoo

#### Odoo 19

```bash
git clone --depth 1 --branch 19.0 https://github.com/odoo/odoo.git odoo
```

#### Odoo 20

```bash
git clone --depth 1 --branch 20.0 https://github.com/odoo/odoo.git odoo
```

The `odoo/` directory is ignored by Git and is therefore not included in the project repository.

### 5. Configuration of the database

```bash
python odoo/odoo-bin \
  --config=./odoo.conf \
  --save
```
### 6. Launching odoo

```bash
python odoo/odoo-bin -c ./odoo.conf
```
