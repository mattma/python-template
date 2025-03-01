# [UV](https://docs.astral.sh/uv/concepts)

- Blazing fast performance with 10-100x speed improvements over pip
- Seamless integration with existing Python packaging standards
- Built-in virtual environment management
- Efficient dependency resolution and lock file support
- Low memory footprint and resource usage

## Getting Start

```bash
# By default, jupyter lab will start the server at http://localhost:8888/lab.
uv run --with jupyter jupyter lab --notebook-dir="notebook"

uv run src/main.py

# Testing
uvx pytest
```

## Manage system python version

This command installs Python 3.13 and, with the --default flag, creates symlinks named python and python3 in your uv-managed binary directory (typically `~/.local/bin`). If you add that directory to the beginning of your PATH, it will override the system Python when you type python globally.

`uv python install 3.13 --default --preview`

command -v python3 finds the full path of the python3 executable. /usr/local/bin/python so that it points to the python3 executable.

`sudo ln -sf "$(command -v python3)" /usr/local/bin/python`

## Start a new project

initializing an empty project using the uv init command

`uv init hello-py`

## Dependencies management

The first time you run the add command, UV creates a new virtual environment in the current working directory and installs the specified dependencies. On subsequent runs, UV will reuse the existing virtual environment and only install or update the newly requested packages, ensuring efficient dependency management.

**UV also updates the pyproject.toml and uv.lock files after each add command.**

Another important process that happens for every add command is resolving dependencies. UV uses a modern dependency resolver that analyzes the entire dependency graph to find a compatible set of package versions that satisfy all requirements. This helps prevent version conflicts and ensures reproducible environments.

```bash
uv add numpy pandas
uv remove pandas

uv add requests=2.1.2 # Installing a specific version
uv add 'requests<3.0.0' # Change the bounds of a package's constraints
uv add 'requests; sys_platform="linux"' # Make a dependency platform-specific
uv add pandas --optional plot excel # Adding optional dependencies
```

### uv.lock

Lock files are an essential part of dependency management in UV. UV manages the lock file automatically. When you run uv add commands to install dependencies, UV automatically generates and updates a `uv.lock` file. This lock file serves several critical purposes:

- It records the exact versions of all dependencies and their sub-dependencies that were installed.
- It ensures reproducible builds by "locking" dependency versions across different environments.
- It helps prevent "dependency hell" by maintaining consistent package versions.
- It speeds up installations since UV can use the locked versions instead of resolving dependencies again.

#### uv.lock vs requirements.txt

While both lock files and `requirements.txt` serve to track dependencies, they have distinct purposes and use cases. Lock files contain detailed information about exact package versions and their complete dependency tree, ensuring consistent environments across development. `Requirements.txt` files are simpler, typically listing only direct dependencies, and are widely supported across Python tools.

Lock files are essential for development to maintain reproducible builds and prevent dependency conflicts. Requirements.txt files are better suited for deployment scenarios or when sharing code with users who may not use UV. They're also necessary for compatibility with tools and services that don't support UV's lock file format.

You can maintain both files by using UV's lock file for development while generating a requirements.txt for deployment. To generate a requirements.txt from a UV lock file, use the following command: `uv export -o requirements.txt`


## Running Python Script

Use the `uv run` command followed by your script name instead of the usual `python hello.py` syntax. The `run` command ensures that the script is executed inside the virtual environment UV created for the project

`un run hello.py`

## managing Python versions in UV

Listing all the existing Python versions UV detects on your system

`uv python list --only-installed`

### [Install Python Versions]((https://docs.astral.sh/uv/concepts/python-versions/#managed-and-system-python-installations))

`uv python install 3.12.x`

### Changing Python versions for the current project

Switch Python versions for your current UV project at any point as long as the new version satisfies the specifications in your **pyproject.toml** file.
you can change the Python version in `.python-version` file to any version above, like 3.11.7. Afterwards, call `uv sync`

```toml
...
requires-python = ">=3.12"
```

The command first checks against existing Python installations. If the requested version isn't found, UV downloads and installs it inside the `${HOME}/.local/share/uv/python` path. UV also creates a new virtual environment inside the project directory, replacing the old one.

This new environment doesn't have the dependencies listed in your pyproject.toml file, so you have to install them with the following command:

`uv pip install -e .`

Note that sometimes uv commands may raise Permission Denied errors. In those cases, be sure to use the sudo command if you are on macOS or Linux or run your command prompt with administrator privileges if you are on Windows. Even better solution would be to change the ownership of the UV home directory to the user:

`sudo chown -R $USER ~/.local/share/uv`


## [UV Tools](https://docs.astral.sh/uv/concepts/tools/)

### Format, Lint, Testing, Type Checking

`uv tool install black flake8 pytest mypy`
`uv tool uninstall black flake8 pytest mypy`
`uv tool upgrade ruff`

Some Python packages are exposed as command-line tools like `black` for code formatting, `flake8` for linting, `pytest` for testing, `mypy` for type checking, etc. UV provides two special interfaces to manage these packages:

`uv tool run black main.py`

Shorthand `uvx black main.py`

## Dependency groups

Dependency groups allow you to organize your dependencies into logical groups, such as development dependencies, test dependencies, or documentation dependencies. This is useful for keeping your production dependencies separate from your development dependencies. To add a dependency to a specific group, use the `--group` flag. users will be able to control which groups to install using the `--group`, `--only-group`, and `--no-group` tags.

`uv add --group group_name package_name`

## Switching From PIP and Virtualenv to UV

If you have an existing project using virtualenv and pip, start by generating a requirements.txt file from your current environment if you haven't already:

```bash
pip freeze > requirements.txt
# Then, create a new UV project in the same directory
uv init .
# Finally, install the dependencies from your requirements file:
uv pip install -r requirements.txt
```

## [Jupyter Notebook](https://docs.astral.sh/uv/guides/integration/jupyter/)

```bash
uv run --with jupyter jupyter lab --notebook-dir="notebook"

# Install packages from within the notebook, we recommend creating a dedicated kernel for your project. Kernels enable the Jupyter server to run in one environment, with individual notebooks running in their own, separate environments.
# To create a kernel, you'll need to install `ipykernel` as a development dependency:
uv add --dev ipykernel

# Then, create the kernel for project with
uv run ipython kernel install --user --env VIRTUAL_ENV $(pwd)/.venv --name=project
# Then, start the server
uv run --with jupyter jupyter lab
```


## Virtual Environment

Create a `.venv` folder for virtual environment.

`uv venv --python 3.13.0`

## uv commands

```bash
# show current project in a tree
uv tree

# sync/install the dependencies with the virtual environment. Update the project's environment
uv sync

# list where UV tool is being installed
uv tool dir

# update the shell in .zshrc and restart shell to apply changes
uv tool update-shell

# publish a project
uv publish --password
```

## Replacing common pip/virtualenv commands

After migration, you can safely remove your old virtualenv directory and start using UV's virtual environment management. The transition is typically seamless, and you can always fall back to pip commands through UV's pip compatibility layer if needed.

| UV equivalent | pip/virtualenv command  |
|---|---|
| uv venv  | python -m venv .venv  |
| uv add package  | python install package  |
| uv pip install -r requirements.txt  | pip install -r requirements.txt  |
| uv remove package  | pip uninstall package  |
| uv pip freeze  | pip freeze |
| uv pip list  | pip list |
