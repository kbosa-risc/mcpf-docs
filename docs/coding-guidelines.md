
# Coding Guidelines and Best Practices

## Basic requirements

If a python function is going to appear in yaml pipeline configuration, its signature and return value must be the following:

```py
from typing import Any
...
def test1(data: dict[str, Any]) -> dict[str, Any]:  
	# specific code part  
	...  
	return data  
``` 

Other functions which are called only from python code do not have to fulfill this requirement.

### Best Practice
To facilitate the general re-usability of your code, use always the same (default) label (e.g.: see DEFAULT_IO_DATA_LABEL in [constants.py](https://github.com/kbosa-risc/mcpf-core/blob/master/mcpf_core/func/constants.py))
for storing the current input data in the passed though dictionary *data*. 

```py
from typing import Any
import mcpf_core.func.constants as constants
...
def function_i(data: dict[str, Any]) -> dict[str, Any]:
	input_data = data[constants.DEFAULT_IO_DATA_LABEL]
	# specific code part
	...
	return data
```

## How to Access Meta Information

By default the dictionary *data* contains a json string called *meta* which may contain the following elements:
* **input_path**,
* **output_path**,
* **tmp_paths**,
* **db_configs** and
* key-value pairs defined in the part **further_configuration** in the yaml configuration.

Additionally, the key value pairs defined by the developer can be also stored here.

### Explicitly access Meta Information

```py
from typing import Any
from mcpf_core.func import constants
import mcpf_core.core.routines as routines
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

### Access Meta Information using Injection

```py
from typing import Any
from mcpf_core.func import constants
...
def func_i(data: dict[str, Any], meta: dict[str, Any]) -> dict[str, Any]:
	value = meta[key]

	# specific code part
	...

	return data
```

When using injection *routines.set_meta_in_data(data, meta)* is called implicitly after the pipeline function has completed execution.

## Configurable Arguments

Additional arguments can be defined in the yaml config file for functions, e.g.:

```yaml
- func_i:
	- {'bool_argument': True }
```

These arguments are added to json string called *meta*, when the corresponding python function called. 
They are stored as a dictionary under the label 'arguments' (use the constants 'ARGUMENTS' defined in [constants.py](https://github.com/kbosa-risc/mcpf-core/blob/master/mcpf_core/func/constants.py)):


### Explicitly access arguments

```py
from typing import Any
from mcpf_core.func import constants
import mcpf_core.core.routines as routines
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

### Access arguments using Injection

```py
from typing import Any
...
def func_i(data: dict[str, Any], arg: dict[str, Any]) -> dict[str, Any]:
	bool_argument = arg['bool_argument']

	# specific code part
	...

	return data
```

**Important remark**: Note that this function throws an exception if `bool_argument` has not been specified in the YAML config file.

### Explicitly handle default arguments

If the argument is missing in the YAML config file, no such key in `arg` exists. Guard against using undefined keys.

```py
from typing import Any
from mcpf_core.func import constants
import mcpf_core.core.routines as routines
...
def func_i(data: dict[str, Any]) -> dict[str, Any]:
	meta = routines.get_meta_data(data)
	if meta[constants.ARGUMENTS]:
		arg = meta[constants.ARGUMENTS]
	bool_argument = arg['bool_argument'] if 'bool_argument' in arg else False

	# specific code part
	...

	routines.set_meta_in_data(data, meta)
	return data
```

### Handle default arguments with decorator

You can use the decorator `@with_default_arguments` to ensure default argument values.

```py
from typing import Any
from mcpf_core.core.decorators import with_default_arguments
...
@with_default_arguments({
	bool_argument: False,
})
def func_i(data: dict[str, Any], arg: dict[str, Any]) -> dict[str, Any]:
	bool_argument = arg['bool_argument'] # will not raise here

	# specific code part
	...

	return data
```


## How loops work in the pipeline 

The main purpose of loops in a (code) pipeline defined in yaml configuration is to execute the same piece of code 
(child-pipeline) on each element of a given list.

### Registering list of data for a loop 

You can register a list of elements on which you would like to execute a loop. 

```py
from typing import Any
import mcpf_core.core.routines as routines
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
once for the members of the loop kernel via the function `routines.pop_iterated_value()`, see below.

**Important remark**: The data type of the enumerated iterator data is required to be a python iterable (it is planned to relax these constrains with numpy arrays and/or pandas series in the future).

### Loop and loop kernel in the yaml configuration

If the framework finds a loop in the yaml configuration (see [Usage](usage.md)), it executes the specified pipeline sequentially on each 
element of the registered list. 
If there is no registered list of iterator values or it is empty, the loop kernel will not be executed at all.  

The framework allows to define embedded loops in a yaml configuration, for instance lets regard the following pipeline which is 
going to list the content of the input folder given in the **input_path** element and of its sub-folders 
(for the the complete yaml configuration, see the [getting started use case](https://github.com/kbosa-risc/mcpf-getting-started/blob/master/README.md)).

```yaml
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

### Accessing the current iterated value within the loop kernel


```py
from typing import Any
import mcpf_core.core.routines as routines
...
def test2_loop_kernel(data: dict[str, Any]) -> dict[str, Any]:
	iterated_value = routines.pop_iterated_value()

	# specific code part
	... # do somethin with `iterated_value`

	return data
```

**Important remark**: For every execution/iteration of a loop kernel the corresponding iterated value is implicitly provided by the framework, but only once. 
Subsequent calls of the *function routines.pop_iterated_value* returns `None`. So it is the responsibility of the developer to make the current iterator value available 
for other functions of the loop kernel, if needed.

### Accessing the current iterated value by injection

```py
from typing import Any
import mcpf_core.core.routines as routines
...
def test2_loop_kernel(data: dict[str, Any], iterated_value: Any) -> dict[str, Any]:
	# specific code part
	... # do somethin with `iterated_value`

	return data
```

### Checking explicitly whether iterated value has been consumed

As noted above within a single loop iteration the iterated value can only be queried once. In order to determine whether the iterated value has already been consumed by a preceding pipeline function, or the iterated value itself happens to be `None`, you can use the function `routines.is_iterated_value_available()`.

```py
from typing import Any
import mcpf_core.core.routines as routines
...
def test2_loop_kernel(data: dict[str, Any]) -> dict[str, Any]:
	if routines.is_iterated_value_available():
		iterated_value = routines.pop_iterated_value()

		# specific code part
		... # do something with `iterated_value`
	else:
		... # iterated_value has already been consumed

	return data
```

### Checking whether iterated value has been consumed with injection

Instead of explicitly calling `routines.is_iterated_value_available()` you can use argument injection.

```py
from typing import Any
...
def test2_loop_kernel(data: dict[str, Any], has_iterated_value: bool, iterated_value: Any) -> dict[str, Any]:
	if has_iterated_value:
		# specific code part
		... # do something with `iterated_value`
	else:
		... # iterated_value has already been consumed

	return data
```

## Generally Implemented Routines

### Best practice using explicit calls

Python functions can be implemented in a general way, such that they can accept either default input, configurable arguments, 
or iterator as input, if any of these are available:

```py
from typing import Any
import mcpf_core.core.routines as routines
from mcpf_core.func import constants
...
def func_with_arguments_given_in_config(data: dict[str, Any]) -> dict[str, Any]:
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
	if routines.is_iterated_value_available():
		iterated_value = routines.pop_iterated_value()
		arg['input'] = iterated_value	# do something with iterated value

	# specific code part
	...

	routines.set_meta_in_data(data, meta)
	return data
```

### Best practice using injection

You can save a lot of boilerplate code if you use argument injection for your pipeline function.

```py
from typing import Any
from mcpf_core.func import constants
from mcpf_core.core.decorators import with_default_arguments
...
@with_default_arguments({
		'input': constants.DEFAULT_IO_DATA_LABEL,
		'output': constants.DEFAULT_IO_DATA_LABEL,
})
def func_with_arguments_given_in_config(data: dict[str, Any], meta: dict[str, Any], has_iterated_value: bool, iterated_value: Any, arg: dict[str, Any]) -> dict[str, Any]:
	# if the function is part of a loop
	if has_iterated_value:
		arg['input'] = iterated_value	# do something with iterated value

	# specific code part
	...
	return data
```

### Already implemented routines

Some generally implemented logics (e.g.: reading/writing csv/parquet files or manipulating dataframes with sql statements, etc) have already been available in the following packages
(please note that these are just preliminary implementations and still have to be finalized):
* [mcpf-io](https://github.com/kbosa-risc/mcpf-io)
* [mcpf-xform](https://github.com/kbosa-risc/mcpf-xform)
* [mcpf-xform-sql](https://github.com/kbosa-risc/mcpf-xform-sql)


