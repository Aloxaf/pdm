# Project file syntax

## Project metadata

There are several differences from the metadata of `setuptools`:

- `readme` is the file name of README file and PDM can derive the content type automatically.
- `author` and `maintainer` is a combination of name and email address in the form of `Name <email>`.

## Package version

You can specify a file source for `version` field like: `version = {from = "pdm/__init__.py"}`, in this form,
the version will be read from the `__version__` variable in that file.

PDM can also read version from SCM tags. If you are using git or hg as the version control system, define the
`version` as follows:

```toml
version = {use_scm = true}
```

## Include and exclude pacakge files

The way of specifying include and exclude files are simple, they are given as a list of glob patterns:

```toml
includes = [
    "**/*.json",
    "mypackage/",
]
excludes = [
    "mypackage/_temp/*"
]
```

If neither `includes` or `excludes` is given, PDM is also smart enough to include top level packages and all data files in them.
Packages can also lie in `src` directory that PDM can find it.

## Dependency specification

### Named requirement

```toml
requests = ">=2.20.0"
pytz = "*"
```

`"*"` means there is no constraint of what version should be used.

### Requirement given by file URL

```toml
pdm = {url="https://github.com/frostming/marko/archive/0.2.6.zip"}
```

### Requirement given by local path

```toml
requests = {path="/path/to/requests"}
```

In this case, the path should be a **directory** on local machine. If you want to install a local **file**,
use `url = "file:///path/to/file` instead.

### VCS requirement

```toml
requests = {git="https://github.com/frostming/marko.git", ref="master"}
```

PDM supports all VCS schemes that are supported by `pip`.

### Editable requirement

Both VCS requirement and local directory requirement can have an `editable = true` flag, meaning it should be installed in editable mode.

### Requirement with markers

```toml
requests = {version=">=2.20.0", marker="os_name!='nt'"}
```

### Extras require

You can have some requirements optional, by putting them under non-default dependency section, as follows:

```toml
[tool.pdm]
extras = ["mysql"]

[tool.pdm.mysql-dependencies]
mysqlclient = "*"
```

Note that the `extras` definition is required, otherwise the requirments is not regarded as the package's requirements. The value is a list of dependency sections that you want to include as extras.

You can also combine the requirements from several sections to an extra require:

```toml
[tool.pdm]
extras = ["mysql", "postgres", "sql=mysql|postgres"]

[tool.pdm.mysql-dependencies]
mysqlclient = "*"

[tool.pdm.postgres-dependencies]
psycopg2 = "*"
```

This `pyproject.toml` produces the equivalent `extras_requires` as in `setup.py`:

```py
extras_require = {
    "mysql": ["mysqlclient"],
    "postgres": ["psycopg2"],
    "sql": ["mysqlclient", "psycopg2"]
}
```

## Console scripts

The following content:

```toml
[tool.pdm.cli]
mycli = "mycli.__main__:main"
```

will be translated to setuptools style:

```python
entry_points = {
    'console_scripts': [
        'mycli=mycli.__main__:main'
    ]
}
```

## Entry points

Other types of entry points are given by `[tool.pdm.entry_points.<type>]` section, with the same
format of `[tool.pdm.cli]` format:

```toml
[tool.pdm.entry_points.pytest11]
myplugin = "mypackage.plugin:pytest_plugin"
```

## Build C extensions

Currently building C extensions still relies on `setuptools`. You should write a python script which contains
a function named `build` and accepts the parameter dictionary of `setup()` as the only argument.
Update the dictionary with your `ext_modules` settings in the function.

Here is an example taken from `MarkupSafe`:

```python
# build.py
from setuptools import Extension

ext_modules = [Extension("markupsafe._speedups", ["src/markupsafe/_speedups.c"])]

def build(setup_kwargs):
    setup_kwargs.update(ext_modules=ext_modules)
```

Now, specify the build script path via `build` in the `pyproject.toml`:

```toml
# pyproject.toml
[tool.pdm]
build = "build.py"
```
