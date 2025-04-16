
# Usage of Minimalist Configurable (Code) Pipeline Framework *(mcpf)*

The mcp framework based on among others the following python modules:
- lasagna,
- toolz,
- json.

## Synopsis

```sh
python3 -m mcpf_core.run [adapted_use_case_configuration.yaml...] basic_use_case_configuration.yaml
```

### Examples

First clone the repository `mcpf-getting-started`.

```sh
git clone https://gitdma.risc-software.at/risc_de/mcp/mcpf-getting-started.git
```

Install the necessary dependencies.

```sh
cd mcpf-getting-started
poetry install
```

Assuming you are in the directory *mcpf-getting-started*:

```sh
python3 -m mcpf_core.run mcp_use_case_getting_started/first_use_case.yaml
```
or, to run pipeline extensions,
```sh
python3 -m mcpf_core.run mcp_use_case_getting_started/first_extension.yaml mcp_use_case_getting_started/first_use_case.yaml
```
	   
## Basic (Use Case) Configuration

### Key Words

* **input_path**: The location of the input data. This path stored in the *meta* data structure, see [Coding Guidelines](coding-guidelines.md#how-to-access-meta-information)
* **output_path**: Expected location of the output data. This path stored in the *meta* data structure, see [Coding Guidelines](coding-guidelines.md#how-to-access-meta-information)
* **tmp_paths**: An enumerated list of locations of intermediate results, if there is any.
* **input_file_name**: input file name or pattern (e.g.:*.tar.gz), if there is any.
* **entry_point**: The label/id of the pipeline, which is the entry point of the defined use case.
* **further_configuration**: key-value pairs defined by the developers, if there is any. They are stored in the *meta* data structure, see [Coding Guidelines](coding-guidelines.md#how-to-access-meta-information)
* **database_configs**: A list of key-value pairs denoting connection data for each type of database any pipeline functions need access to. Then mandatory key is `type` which discriminates the database type the configuration is for. See [MCPF Database access](https://gitdma.risc-software.at/risc_de/mcp/mcpf-db/-/blob/master/README.md) for more information.
* **imports**: An enumerated list of python modules (given with absolute or relative path) which contain the functions directly listed/called in the parts *pipelines* or *pipeline_extension* of the YAML configurations.
* **pipelines**: A list of pipelines. A pipeline is a labeled list of other pipelines, python functions or, as special element, any word starting with the prefix `loop`. The syntax of elements of a pipeline is the following
  (for more information see [Coding Guidelines](coding-guidelines.md#how-loops-work-in-the-pipeline)):  
  ```yaml
  - label_k:												# name of the pipeline
	- label_m: ~												# reference to another pipeline, the placeholder ~ (tilde) is a required in this case 
	- function_i: ~											# python function without any additional argument, the placeholder ~ (tilde) is a required in this case
	- function_j: 											# python function with additional arguments
		{string_arg1: 'some_string', bool_arg2: True, ...}
	- loop: label_n											# a loop which iteratively executes the pipeline identified with `label_n'
  ```  

**Important remark**: In the current preliminary implementation of the mcp framework every python function listed in the yaml configuration must have a unique name, regardless of whether
they are defined in different modules.

For particular YAML configuration examples, see [Getting Started](https://gitdma.risc-software.at/risc_de/mcp/mcpf-getting-started/-/blob/master/README.md).

## Adapted (Use Case) Configuration

In adapted configuration you can overwrite all the parts of the basic/previously adapted configuration, except the part initiated with the key word *pipelines*. Instead of the key word *pipelines* you can use *pipeline_extension*.
In the section `pipeline_extension` of the configuration you can redefine any part/child-pipeline given in the basic configuration by reusing its name. For particular examples see
[Getting Started](https://gitdma.risc-software.at/risc_de/mcp/mcpf-getting-started/-/blob/master/README.md)

> **_NOTE:_** If you overwrite a pipeline called `pipeline_x` within a `pipeline_extension` block, **every** item of `pipeline_x` will be thrown away. You cannot "inherit" items from the basic pipeline definition, you have to completely redefine them.
