
# Usage of Minimalist Configurable (Code) Pipeline *(mcp)* Framework

The mcp framework based on among others the following python modules:
- lasagna,
- toolz,
- json.

For more information and for the additional requirements of the use cases see the file [pyproject.toml](../pyproject.toml)

## Synopsis

>      pipeline_runtime.py  
>         [adapted_use_case_configuration.yaml] basic_use_case_configuration.yaml

### Examples

Assuming you are in the directory *mcp_frm*:

```
./pipeline_runtime.py ../mcp_use_case_fronius/fronius_use_case1.yaml
```
or
```
./pipeline_runtime.py  
	       ../mcp_use_case_fronius/fronius_use_case2.yaml ../mcp_use_case_fronius/fronius_use_case1.yaml
```
	   
## Basic (Use Case) Configuration

### Key Words

* **input_path**: The location of the input data. This path stored in the *meta* data structure, see [../mcp_general_functions/README.md](../mcp_general_functions/README.md)
* **output_path**: Expected location of the output data. This path stored in the *meta* data structure, see [../mcp_general_functions/README.md](../mcp_general_functions/README.md)
* **tmp_paths**: An enumerated list of locations of intermediate results, if there is any.
* **input_file_name**: input file name or pattern (e.g.:*.tar.gz), if there is any.
* **entry_point**: The label/id of the pipeline, which is the entry point of the defined use case.
* **further_configuration**: key-value pairs defined by the developers, if there is any. They are stored in the *meta* data structure, see [../mcp_general_functions/README.md](../mcp_general_functions/README.md)
* **database_configs**: It describes itself. For detailed information see [../mcp_use_case_sim2influx/sim2influx_use_case.yaml](../mcp_use_case_sim2influx/sim2influx_use_case.yaml) and [../mcp_use_case_sim2influx/sim2timescale_use_case.yaml](../mcp_use_case_sim2influx/sim2timescale_use_case.yaml)
* **imports**: An enumerated list of python modules (given with absolute or relative path), which contains the functions directly listed/called in the parts *pipelines* or *extended_pipeline* of the yaml configurations.
* **pipelines**: A list of pipelines. A pipeline is a labeled list of other pipelines, python functions or as special element any word starting with the prefix *loop*. The syntax of elements of a pipeline is the following
  (for more information see [../mcp_general_functions/README.md](../mcp_general_functions/README.md)):  
  ```
  - label_k:												# name of the pipeline
	- label_m: ~												# reference to another pipeline, the placeholder ~ (tilde) is a required in this case 
	- function_i: ~											# python function without any additional argument, the placeholder ~ (tilde) is a required in this case
	- function_j: 											# python function with additional arguments
		{string_arg1: 'some_string', bool_arg2: True, ...}
	- loop: label_n											# a loop which iteratively executes the pipeline identified with `label_n'
  ```  

**Important remark**: In the current preliminary implementation of the mcp framework every python function listed in the yaml configuration must have a unique name, regardless of whether
they are defined in different modules.

For particular yaml configuration examples, see the use cases.

## Adapted (Use Case) Configuration

In adapted configuration you can overwrite all the parts of the basic/previously adapted configuration, except the part initiated with the key word *pipelines*. Instead of the key word *pipelines* you can use *pipeline_extension*.
In the section 'pipeline_extension' of the configuration you can redefined any part/child-pipeline given in the basic configuration by reusing its name. For particular examples see:

- [../mcp_use_case_fronius/fronius_use_case2.yaml](../mcp_use_case_fronius/fronius_use_case2.yaml),
- [../mcp_use_case_UPO/upo_use_case2.yaml](../mcp_use_case_UPO/upo_use_case2.yaml) or
- [../mcp_use_case_sim2influx/sim2timescale_use_case.yaml](../mcp_use_case_sim2influx/sim2timescale_use_case.yaml)
