
# Coding Guidelines and Best Practices

## Basic requirements

If a python function is going to appear in yaml pipeline configuration, its signature and return value must be the following:

```
from typing import Any
...
def test1(data: dict[str, Any]) -> dict[str, Any]:  
	# specific code part  
	...  
	return data  
``` 

Other functions, which are called only from python code should not fulfill this requirement.

### Best Practice
To facilitate the general re-usability of your code, use always the same (default) label (e.g.: see DEFAULT_IO_DATA_LABEL in [constants.py](constants.py) 
for storing the current input data in the passed though dictionary *data*. 

```
from typing import Any
import mcp_general_functions.constants as constants
...
def function_i(data: dict[str, Any]) -> dict[str, Any]:
	input_data = data[constants.DEFAULT_IO_DATA_LABEL]
	# specific code part
	...
	return data
```

## How to Access to the Meta Information

By default, the dictionary *data* contains a json string called *meta* which may contain the following elements:
* **input_path**,
* **output_path**,
* **tmp_paths**,
* **db_configs** and
* key-value pairs defined in the part **further_configuration** in the yaml configuration.

Additionally, the key value pairs defined by the developer can be also stored here. 

```
from typing import Any
from mcp_general_functions import constants
import mcp_frm.pipeline_routines as routines
...
def func_i(data: dict[str, Any]) -> dict[str, Any]:
	meta = routines.get_meta_data(data)
	value = meta[key]

	# specific code part
	...

	routines.set_meta_in_data(data, meta)
	return data
```

**Important remark**: The content of the meta can be updated from the code, but in such a case the call *routines.set_meta_in_data(data, meta)* must be executed 
before the end of the function, otherwise the changes are not stored. 

## Configurable Arguments

Additional arguments can be defined in the yaml config file for functions, e.g.:

```
- func_i:
	- {'bool_argument': True }
```

These arguments are added to json string called *meta*, when the corresponding python function called. 
They are stored as a dictionary under the label 'arguments' (use the constants 'ARGUMENTS' defined in [constants.py](constants.py)):

```
from typing import Any
from mcp_general_functions import constants
import mcp_frm.pipeline_routines as routines
...
def func_i(data: dict[str, Any]) -> dict[str, Any]:
	meta = routines.get_meta_data(data)
	if meta[constants.ARGUMENTS]:
		arg = meta[constants.ARGUMENTS]
	bool_argument = arg['bool_argument']

	# specific code part
	...

	routines.set_meta_in_data(data, meta)
	return data
```

**Important remark**: Always access to the content of *meta data* only via the provided *routines.get_meta_data* function otherwise the 
arguments specified for the current function in the yaml config file will not be set.

## How loops work in the pipeline 

The main purpose of loops in a (code) pipeline defined in yaml configuration is to execute the same piece of code 
(child-pipeline) on each elements of a given list.

### Registering list of data for a loop 

You can register a list of elements, on which you would like to execute a loop. 

```
from typing import Any
import mcp_frm.pipeline_routines as routines
...
def function_i(data: dict[str, Any]) -> dict[str, Any]:
	# specific code part
	...

	loop_list = list(range(0,10))
	routines.register_loop_iterator_list(loop_list)
	# or
	routines.register_loop_iterator_list(loop_list, deep_copy = True)
	# specific code part
	...
	
	return data
```

If sooner or later a loop will come in the pipeline configuration after this step above, 
then the loop will go through all the elements of the given list. The framework will provide each element of the list 
once for the members of the loop kernel via the function routines.pop_loop_iterator(data), see below.

**Important remark**: The data type of the enumerated iterator data required to be python list (it is planned to relax these constrains with numpy arrays and/or pandas series in the future).

### Loop and loop kernel in the yaml configuration

If the framework finds a loop in the yaml configuration (see [../mcp_frm/README.md](../mcp_frm/README.md)), it executes the specified pipeline sequentially on each 
element of the registered list. 
If there is no registered list of iterator values or it is empty, the loop kernel will not be executed at all.  

The framework allows to define embedded loops in a yaml configuration, for instance lets regard the following pipeline which is 
going to list the content of the input folder given in the **input_path** element and of its sub-folders 
(for the complete yaml configuration, see the [getting started use case](../mcp_use_case_getting_started/README.md)).

```
pipelines:
  - main_p:
      - list_dir:
          - { relative_path: True, output_for_iteration: True }
      - loop: list_input_dirs_p

  - list_input_dirs_p:
      - processing_files_p: ~
      - list_dir:
          - { 'only_file_names_return': True, 'output_for_iteration': True }
      - loop: processing_files_p

  - processing_files_p:
      - print_to_stdout: ~
```

### Accessing to the current iterator value within the loop kernel


```
from typing import Any
import mcp_frm.pipeline_routines as routines
...
def test2_loop_kernel(data: dict[str, Any]) -> dict[str, Any]:
	iterator = routines.pop_loop_iterator()

	# specific code part
	...

	return data
```

**Important remark**: For every execution/iteration of a loop kernel the corresponding iterator value is implicitly provided by the framework, but only once. 
Subsequent calls of the *function routines.pop_loop_iterator* returns with 'None' value. So it is the responsibility of the developer to make the current iterator value available 
for other functions of the loop kernel, if needed.

## Generally Implemented Routines

### Best practice

Python functions can be implemented in a general way, such that they can accept either default input, configurable arguments, 
or iterator as input, if any of these are available:

```
from typing import Any
import mcp_frm.pipeline_routines as routines
from mcp_general_functions import constants
...
def func_with_arguments_given_in_config(data: dict[str, Any]) -> dict[str, Any]:
	iterator = routines.pop_loop_iterator()
	meta = routines.get_meta_data(data)
	# default_arguments_values
	arg = {
		'input': constants.DEFAULT_IO_DATA_LABEL,
		'output': constants.DEFAULT_IO_DATA_LABEL,
	}
	# merging default values with current argument values
	if meta[constants.ARGUMENTS]:
		arg = arg | meta[constants.ARGUMENTS]
	# if the function part of a loop
	if iterator:
		arg['input'] = iterator

	# specific code part
	...

	routines.set_meta_in_data(data, meta)
	return data
```

### Already implemented routines

Some generally implemented logics (e.g.: reading/writing csv/parquet files or accessing influx/timescale database) have already been available in the following source files
(please note that these are just preliminary implementations and still have to be finalized):
* [mcp_general_io.py](mcp_general_io.py)
* [mcp_general_db.py](mcp_general_db.py)
* [mcp_general_transformations.py](mcp_general_transformations.py)


