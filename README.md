
# Minimalist Configurable (Code) Pipeline Framework

The Minimalist Configurable (Code) Pipeline Framework (MCPF) uses one or more YAML configuration files which define pipelines.

Each pipeline consists of a list of python functions that are called in the very order specified.

* See [usage](docs/usage.md) on how to use the MCPF.
* See [coding guidelines](docs/coding-guidelines.md) on how to write your own pipeline functions in Python.

A preliminary version of a [DSL schema](dsl_schema/mcpf.schema.json) (in JSON format) is also provided and can be integrated into development environments such as Visual Studio Code or PyCharm. This enables syntax validation, auto-completion, and parameter hints during pipeline authoring. See the [README](dsl_schema/README) for details.

## Extras

The MCPF is build on a modular collection of packages. This package provides the following extras to facilitate the installation of your specific MCPF needs.

* `io` IO-related functions with filesystem and Pandas dataframe support.
* `xform` transformation functions to convert, filter and process data in Pandas dataframes.
* `xql` Query pandas dataframes with an SQL like language.
* `postgres` PostgreSQL related functions.
* `influx` Influx related functions.

To install extras, use

```sh
poetry add 'mcpf[postgres]'
```
