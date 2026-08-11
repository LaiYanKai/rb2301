9a&emsp;Advanced Parameters (Same Node)
==============

***RB2301 Robot Programming***

**&copy; Lai Yan Kai, National University of Singapore**

This chapter is all about declaring, getting and setting ROS parameters, and related concepts such as prefixes, depth, and types.
ROS2 CLI and client library (Python) examples are also shown.

# Table of Contents
[1&emsp;Parameter Concepts](#1parameter-concepts)

&emsp;[1.1&emsp;Declare, Get, and Set](#11declare-get-and-set)

&emsp;[1.2&emsp;Types](#12types)

&emsp;[1.3&emsp;Prefixes and Depth](#13prefixes-and-depth)

[2&emsp;Command Line Interface](#2command-line-interface)

[3&emsp;Client Library (Python)](#3client-library-python)

&emsp;[3.1&emsp;Declare Parameter](#31declare-parameter)

&emsp;[3.2&emsp;Get Parameter](#32get-parameter)

&emsp;[3.3&emsp;Set Parameter](#33set-parameter)

&emsp;[3.4&emsp;Set Parameters Callback](#34set-parameters-callback)

&emsp;[3.5&emsp;Tasks](#35tasks)

[4&emsp;The YAML File](#4the-yaml-file)

&emsp;[4.1&emsp;Writing a YAML File](#41writing-a-yaml-file)

&emsp;[4.2&emsp;Where to Place](#42where-to-place)

&emsp;[4.3&emsp;Installing with `setup.py`](#43installing-with-setuppy)

&emsp;[4.4&emsp;Loading with `ros2 run`](#44loading-with-ros2-run)

&emsp;[4.5&emsp;Loading with Launch File](#45loading-with-launch-file)

&emsp;[4.6&emsp;Tasks](#46tasks)


# 1&emsp;Parameter Concepts

## 1.1&emsp;Declare, Get, and Set
Parameters are <b>unique to the nodes</b> that <b>declared</b> them. 
The declarations are to inform the node that the parameters will be used in the node's lifetime. 
The parameters can then be read (<b>get</b>) or written (<b>set</b>).

## 1.2&emsp;Types
Parameters have types.
Each type is associated with an enum value. 
The table below shows the enum associated with the `set_parameters()` function used in [3.3 Set Parameter](#33set-parameter). 

| Type | Example Syntax | Enum |
|-|-|-|
| **Boolean** | `True` or `False` | `Param.Type.BOOL` \* |
| **Integer** | `1` | `Param.Type.INTEGER` \* |
| **Double** | `1.0` | `Param.Type.DOUBLE` \* |
| **String** | `'hi'` | `Param.Type.STRING` \* |
| **Byte array** | `bytes([0x1A, 0x00, 0x10])` | `Param.Type.BYTE_ARRAY` \* |
| **Boolean array** | `[True`, `False`, `True]` | `Param.Type.BOOL_ARRAY` \* |
| **Integer array** | `[1, 2, 3]` | `Param.Type.INTEGER_ARRAY` \* |
| **Double array**  | `[1.0, 2.1, -3.0]` | `Param.Type.DOUBLE_ARRAY` \* |
| **String array** | `['hi', 'i', 'am']` | `Param.Type.STRING_ARRAY` \* |

\* `Param` is imported below. 
The renamed class `Param` is meant to differentiate from the similarly named class `rcl_interfaces/msg/Parameter`.
```python
from rclpy import Parameter as Param
```

## 1.3&emsp;Prefixes and Depth
Parameters can be **prefixed** (grouped together) based on their similarities. Groups can be placed under another group (nested), resulting in deeper and deeper groups. Each group is delimited with the period `.`.
A parameter's **depth** depends on how deeply prefixed a parameter is. 
Take note that the **full parameter name includes the prefix**.

The following example shows the full name, prefix and depth of every parameter.
```python
has_image: True
image.path: 'path/to/img.jpg'
image.origin.x: 125
image.origin.y: 100
```
- The parameter `has_image` has no prefix and is at depth 1. 
- The parameter `image.path` has a `image` prefix and is at depth 2. 
- Parameters `image.origin.x` and `image.origin.y` have a `image.origin` prefix and is at depth 3.

# 2&emsp;Command Line Interface

The `ros2 param` CLI is used to modify parameters. The following table found in [03_CLI.md](03_CLI.md#7parameters) shows the `ros2 param` CLI. 

| | Command | Description |
| - | - | - |
| 1. | `ros2 param delete /node_a a_dyn_param` | Deletes a dynamically generated parameter `a_dyn_param` from the node `/node_a`. Rarely used. |
| 2. | `ros2 describe /node_a param_a` | Describes the data type, constraints and information about the parameter `param_a` from the `/node_a` node. |
| 3. | `ros2 param dump /node_a` | Prints out all the parameters from a node `/node_a` in YAML file format. |
| 4. | `ros2 param get /node_a param_a` | Gets the value in the parameter `param_a` from the node `/node_a`. |
| 5. | `ros2 param list` | Lists all parameters from all existing nodes. |
| 6. | `ros2 param load /node_a a_path_to_yaml_file` | Opens a YAML file containing parameters for the node `/node_a` at the path `a_path_to_yaml_file` and loads them into the node. The YAML file can contain parameters for other nodes as well. | 
| 7. | `ros2 param set /node_a param_a value_a` | Sets a parameter `param` in the node `/node_a` with the value `value_a`. The value is YAML-interpreted, so if `value_a` is `true` or `on`, the value is interpreted as the boolean `true`. If the parameter needs to accept the string `on` instead of a boolean value, use `"'on'"` instead. |

The following table shows how to set a node's parameters using `ros2 run`:

| | Command | Description |
| - | - | - |
| 1. | `ros2 run pkg_a node_a --ros-args -p prm_a:=0` | Sets the integer parameter `prm_a` to 0. |
| 2. | `ros2 run pkg_a node_a --ros-args -p grp_a.prm_a:=0 ` | Sets the integer parameter `grp_a.prm_a` to 0. The parameter has the prefix `grp_a`. |
| 3. | `ros2 run pkg_a node_a --ros-args -p grp_a.prm_a:=0 -p prm_b:='a string'` | Sets two parameters, delimited by `-p`. |

# 3&emsp;Client Library (Python)

This section shows the Python code to get and set parameters from the same node, and assign a callback that is called immediately before the parameters are set.

The following Pythonic template from [05_Node.md](05_Node.md) is used:
```python
# 1. IMPORT
import rclpy
from rclpy.node import Node

# NODE CLASS
class NodeA(Node):
    def __init__(self):
        super().__init__('node_a')
        # 2. NODE PROPERTIES

        # 3. NODE HANDLES

    # 4. NODE CALLBACKS

    # 5. HOW TO USE 

# MAIN BOILER PLATE
def main(args=None):
    rclpy.init(args=args)
    node = NodeA()
    rclpy.spin(node)
    rclpy.shutdown()
if __name__ == '__main__':
    main()
```
## 3.1&emsp;Declare Parameter
The following table shows how to declare a parameter.
<table><tbody>
    <tr>
        <th></th>
        <th>Append To</th>
        <th>Code</th>
    </tr>
    <tr>
        <td>1.</td>
        <td><b>Import</b></td>
        <td>If the descriptor is needed, <pre lang="python">from rcl_interfaces.msg import ParameterDescriptor</pre></td>
    </tr>
    <tr>
        <td>3.</td>
        <td><b>Node Constructor</b></td>
        <td>
            Declare the parameter, so the node knows that it needs to access this parameter in its life time.
            <ul>
                <li><code>'grp_a.prm_a'</code>: The name of the parameter </li>
                <li><code>default_value</code>: The initial value. The type of the parameter is implicitly specified by this value, so it is important to use the correct type here.
                See the example syntax in <a href="#13types">1.3 Types</a>.</li>
                <li><code>description</code>: An optional description specified using the <code>rcl_interfaces/msg/ParameterDescriptor</code> class.
            </ul>
<pre lang="python">
self.declare_parameter('grp_a.prm_a', default_value)
# or
desc_b = ParameterDescriptor(description="a description") 
self.declare_parameter('grp_a.prm_b', default_value, desc_a)
# see example below for more options in ParameterDescriptor
</pre>
        The function <code>declare_parameters()</code> (not shown) can also be used to set multiple parameters at once.
        </td>
    </tr>
</tbody></table>

The example below shows how to declare two parameters, one with the description, and the other without.
```python
# 1. IMPORT
import rclpy
from rclpy.node import Node
from rcl_interfaces.msg import ParameterDescriptor, IntegerRange

# NODE CLASS
class NodeA(Node):
    
    def __init__(self):
        super().__init__('node_a')
        desc_blue = ParameterDescriptor(
            description = 'the color blue, constrained',
            additional_constraints = 'some help text for some other constraint',
            integer_range = [
                IntegerRange(
                    from_value = 0,
                    to_value = 200,
                    step = 2
                )
            ]
            # `name=` is ignored by declare_parameter()
            # `type=` is ignored by declare_parameter()
        )
        self.declare_parameter('color.blue', 10, desc_blue) # int
        self.declare_parameter('color.red', 60) # int

# MAIN BOILER PLATE
def main(args=None):
    rclpy.init(args=args)
    node = NodeA()
    rclpy.spin(node)
    rclpy.shutdown()
if __name__ == '__main__':
    main()
```

## 3.2&emsp;Get Parameter
The following table shows how to get a parameter's value.

<table><tbody>
    <tr>
        <th></th>
        <th>Append To</th>
        <th>Code</th>
    </tr>
    <tr>
        <td>5.</td>
        <td><b>How to Use</b></td>
        <td>
            The value can be obtained with:
<pre lang="python">
def in_some_function(self):
    value_a = self.get_parameter_value('grp_a.prm_a').value
</pre>
        </td>
    </tr>
</tbody></table>

The example is built upon the previous example, and shows how to get the values of the parameters in a timer callback.

```python
# 1. IMPORT
import rclpy
from rclpy.node import Node
from rcl_interfaces.msg import ParameterDescriptor, IntegerRange

# NODE CLASS
class NodeA(Node):
    
    def __init__(self):
        super().__init__('node_a')
        
        desc_blue = ParameterDescriptor(
            description = 'the color blue, constrained',
            additional_constraints = 'some help text for some other constraint',
            integer_range = [
                IntegerRange(
                    from_value = 0,
                    to_value = 200,
                    step = 2
                )
            ]
            # `name=` is ignored by declare_parameter()
            # `type=` is ignored by declare_parameter()
        )
        self.declare_parameter('color.blue', 10, desc_blue) # int
        self.declare_parameter('color.red', 60) # int

        self.timer = self.create_timer(0.5, self.timer_callback)

    # 4. NODE CALLBACKS
    def timer_callback(self):
        # 5. HOW TO USE 
        blue_value = self.get_parameter('color.blue').value
        print(f'Got Blue: {blue_value}')

        red_value = self.get_parameter('color.red').value
        print(f'Got Red: {red_value}')

# MAIN BOILER PLATE
def main(args=None):
    rclpy.init(args=args)
    node = NodeA()
    rclpy.spin(node)
    rclpy.shutdown()
if __name__ == '__main__':
    main()
```

## 3.3&emsp;Set Parameter
The following table shows how to set the value for one or multiple parameters.

<table><tbody>
    <tr>
        <th></th>
        <th>Append To</th>
        <th>Code</th>
    </tr>
    <tr>
        <td>1.</td>
        <td><b>Import</b></td>
        <td>The following class is renamed because there is a message <code>rcl_interfaces/msg/Parameter</code> that may be used.<pre lang="python">from rclpy import Parameter as Param</pre></td>
    </tr>
    <tr>
        <td>5.</td>
        <td><b>How to Use</b></td>
        <td>
            Use the <code>Param</code> class to specify the value.
            <ul>
                <li><code>'grp_a.prm_a'</code>: The name of the parameter.</li>
                <li><code>Param.Type.INTEGER</code>: Replace this with the correct type from the table in <a href="#13types">1.3&emsp;Types</a></li>
                <li><code>5</code>: Replace this with the value to use. Must be the correct type.</li>
            </ul>
            Then use the <code>set_parameters()</code> function to set the parameter.
<pre lang="python">
def in_some_function(self):
    new_prm_a = Param(
        'grp_a.prm_a',
        Param.Type.INTEGER,
        5
    )
self.set_parameters([new_prm_a])
</pre>
            To set multiple parameters at one go, simply extend the list in <code>set_parameters()</code>:
<pre lang="python">
def in_some_function(self):
    new_prm_a = Param(
        'grp_a.prm_a',
        Param.Type.INTEGER,
        5
    )    
    new_prm_b = Param(
        'prm_b',
        Param.Type.DOUBLE,
        5.0
    )
    self.set_parameters([new_prm_a, new_prm_b])
</pre>
        </td>
    </tr>
</tbody></table>

The example is built upon the example in the previous section, and the parameters are set based on their current value.

```python
# 1. IMPORT
import rclpy
from rclpy.node import Node
from rcl_interfaces.msg import ParameterDescriptor, IntegerRange
from rclpy import Parameter as Param 

# NODE CLASS
class NodeA(Node):
    
    def __init__(self):
        super().__init__('node_a')
        
        desc_blue = ParameterDescriptor(
            description = 'the color blue, constrained',
            additional_constraints = 'some help text for some other constraint',
            integer_range = [
                IntegerRange(
                    from_value = 0,
                    to_value = 200,
                    step = 2
                )
            ]
            # `name=` is ignored by declare_parameter()
            # `type=` is ignored by declare_parameter()
        )
        self.declare_parameter('color.blue', 10, desc_blue) # int
        self.declare_parameter('color.red', 60)

        self.timer = self.create_timer(0.5, self.timer_callback)

    # 4. NODE CALLBACKS
    def timer_callback(self):
        # 5. HOW TO USE 

        blue_param_value = self.get_parameter('color.blue').value
        new_blue_param = Param(
            'color.blue',
            Param.Type.INTEGER,
            (blue_param_value + 50) % 200
        )

        red_param_value = self.get_parameter('color.red').value
        new_red_param = Param(
            'color.red',
            Param.Type.INTEGER,
            (red_param_value + 5) % 255
        )

        print(f'Try setting Blue({new_blue_param.value:3d}) Red({new_red_param.value:3d})')

        new_params = [
            new_blue_param,
            new_red_param
        ]
        self.set_parameters(new_params)

# MAIN BOILER PLATE
def main(args=None):
    rclpy.init(args=args)
    node = NodeA()
    rclpy.spin(node)
    rclpy.shutdown()
if __name__ == '__main__':
    main()
```

## 3.4&emsp;Set Parameters Callback
The following table shows how to assign a callback immediately before parameters are set.

The callback can be used to copy a parameter value like in topic callbacks.
The difference is that topic callbacks process one message at a time, while parameter callbacks can process multiple parameters at once.

The callback can return with a failure, in which case all of the listed parameters would not be set. This can occur if the developer decides that the value contained in a parameter is not invalid.

<table><tbody>
    <tr>
        <th></th>
        <th>Append To</th>
        <th>Code</th>
    </tr>
    <tr>
        <td>1.</td>
        <td><b>Import</b></td>
        <td><pre lang="python">from rcl_interfaces.msg import SetParametersResult</pre></td>
    </tr>
    <tr>
        <td>3.</td>
        <td><b>Node Constructor</b></td>
        <td>
            Associate the callback <code>set_parameters_callback()</code> in the constructor.
<pre lang="python">
self.add_on_set_parameters_callback(self.set_parameters_callback)
</pre>
        </td>
    </tr>
    <tr>
        <td>4.</td>
        <td><b>Node Callbacks</b></td>
        <td>
            The value can be obtained with:
<pre lang="python">
def set_parameters_callback(self, parameter_list):
    for parameter in parameter_list:
        if parameter.name == 'grp_a.prm_a':
            print(f'Do something with parameter.value')
    return SetParametersResult(successful=True)
</pre>
        </td>
    </tr>
</tbody></table>

The following example is built upon the previous example. A parameter callback is assigned, which simply prints out the newly set values.

```python
# 1. IMPORT
import rclpy
from rclpy.node import Node
from rcl_interfaces.msg import ParameterDescriptor, IntegerRange
from rclpy import Parameter as Param 

# NODE CLASS
class NodeA(Node):
    
    def __init__(self):
        super().__init__('node_a')
        
        desc_blue = ParameterDescriptor(
            description = 'the color blue, constrained',
            additional_constraints = 'some help text for some other constraint',
            integer_range = [
                IntegerRange(
                    from_value = 0,
                    to_value = 200,
                    step = 2
                )
            ]
            # `name=` is ignored by declare_parameter()
            # `type=` is ignored by declare_parameter()
        )
        self.declare_parameter('color.blue', 10, desc_blue) # int
        self.declare_parameter('color.red', 60)

        self.timer = self.create_timer(0.5, self.timer_callback)
        
        self.add_on_set_parameters_callback(self.set_parameters_callback)

    # 4. NODE CALLBACKS
    def set_parameters_callback(self, parameter_list):
        for parameter in parameter_list:
            if parameter.name == 'color.blue':
                print(f'Setting color.blue: {parameter.value}')
            elif parameter.name == 'color.red':
                print(f'Setting color.red: {parameter.value}')            

        return SetParametersResult(successful=True)

    def timer_callback(self):
        # 5. HOW TO USE 
        blue_param_value = self.get_parameter('color.blue').value
        new_blue_param = Param(
            'color.blue',
            Param.Type.INTEGER,
            (blue_param_value + 50) % 200
        )

        red_param_value = self.get_parameter('color.red').value
        new_red_param = Param(
            'color.red',
            Param.Type.INTEGER,
            (red_param_value + 5) % 255
        )

        print(f'Try setting Blue({new_blue_param.value:3d}) Red({new_red_param.value:3d})')

        new_params = [
            new_blue_param,
            new_red_param
        ]
        self.set_parameters(new_params)

# MAIN BOILER PLATE
def main(args=None):
    rclpy.init(args=args)
    node = NodeA()
    rclpy.spin(node)
    rclpy.shutdown()
if __name__ == '__main__':
    main()
```

## 3.5&emsp;Tasks

1. Design a node called `parameters` that can be run with `ros2 run rb2301_tutorial prms`.

2. In the node constructor, declare the following parameters with the following initial values:
    
    | Parameter | Type | Initial Value |
    |-|-|-|
    | `values.bool_v` | Boolean. | `False`. |
    | `values.int_v` | Integer. | `0`. |
    | `values.dbl_v` | Double. | `0.0`. |
    | `values.str_v` | String. | `''`. |
    | `bool_arr_v` | Boolean array. | One-element list containing `False` |
    | `int_arr_v` | Integer array. | One-element list containing `0`. |
    | `dbl_arr_v` | Double array. | One-element list containing `0.0`. | 
    | `str_arr_v` | String array. | One-element list containing an empty string `''`. |
    
    **[Q1a]** What is the code to declare these parameters?
    
3. Design a set parameters callback that prints out every new value to be set, using the print statement below. Replace `prm_name` with the parameter name, and `prm_value` with the newly set value:

    ```python
    print(f'Setting "{prm_name}": {prm_value}')
    ```
    
    **[Q1b]** What is the code assign a callback in the node constructor?

    **[Q1c]** What is the code for the callback? Include the line containing the function declaration keyword `def` and the body of the callback.

        
4. Design a timer callback that runs every 1 second, printing out the current value of the `values.str_v` parameter. 
Use the ROS logger function below by replacing `values.str_v` with the value stored in the parameter.

    ```python
    self.get_logger().info(f'{str_v_value}')
    ```

    **[Q1d]** What is the code for the timer callback? Include the function declaration and body. Do not include the initialization in the node constructor.

    **[Q1e]** What is the `ros2 param` CLI to set `values.str_v` to the string `'string with spaces'` while the node is running?

    **[Q1f]** What is the `ros2 run` CLI to initialize `values.str_v` to the string `'overwritten string'`?
    

# 4&emsp;The YAML File
The YAML file allows parameters to be bulk-loaded to several nodes at once. The file can be used in the launch file, or from the `ros2 run` CLI.


## 4.1&emsp;Writing a YAML File
The following template demonstrates the minimal rules for writing a parameter YAML file:
```yaml
namespace_a:                # namespace if any
    node_a:                 # the name of the node in the namespace
        ros__parameters:    # always put this under the node.
            grp_a:          # break down prefixes into their groups.
                param_a: 0  # int parameter example
            param_b: 0      # int parameter examp,e
```

The following shows an example of a YAML file and a table breaking down the example.

```yaml
namespace_a: 
    node_a: 
        ros__parameters: 
            color:
                red: 0  
                blue: 0
                green: 0
            grp_a:
                grp_b:
                    prm_a: 0.0
            bool_a: True
            integer_a: 0
            double_a: 0.0
            string_a: 'a string'
            bool_array_a: [True, False]
            integer_array_a: [1,2,3]
            double_array_a: [-1.1, -2.2, -3.3]
            string_array_a: ['a', 'b', 'c']

node_b:
    ros__parameters:
        integer_a: 0
```

<table><tbody>
    <tr>
        <th>Node</th>
        <th>Parameter</th>
        <th>Prefix</th>
        <th>Type</th>
        <th>Value</th>
    </tr>
    <tr>
        <td rowspan="12"><code>namespace_a/node_a</code></td>
        <td><code>color.red</code></td>
        <td><code>color</code></td>
        <td>Integer</td>
        <td><code>0</code></td>
    </tr>
    <tr>
        <td><code>color.blue</code></td>
        <td><code>color</code></td>
        <td>Integer</td>
        <td><code>0</code></td>
    </tr>
    <tr>
        <td><code>color.green</code></td>
        <td><code>color</code></td>
        <td>Integer</td>
        <td><code>0</code></td>
    </tr>
    <tr>
        <td><code>grp_a.grp_b.prm_a<code></td>
        <td><code>grp_a.grp_b</code></td>
        <td>Double</td>
        <td><code>0.0</code></td>
    </tr>
    <tr>
        <td><code>bool_a</code></td>
        <td>&mdash;</td>
        <td>Boolean</td>
        <td><code>True</code></td>
    </tr>
    <tr>
        <td><code>integer_a</code></td>
        <td>&mdash;</td>
        <td>Integer</td>
        <td><code>0</code></td>
    </tr>
    <tr>
        <td><code>double_a</code></td>
        <td>&mdash;</td>
        <td>Double</td>
        <td><code>0.0</code></td>
    </tr>
    <tr>
        <td><code>string_a</code></td>
        <td>&mdash;</td>
        <td>String</td>
        <td><code>'a string'</code></td>
    </tr>
    <tr>
        <td><code>bool_array_a</code></td>
        <td>&mdash;</td>
        <td>Boolean Array</td>
        <td>Array containing booleans <code>True</code> and <code>False</code></td>
    </tr>
    <tr>
        <td><code>integer_array_a</code></td>
        <td>&mdash;</td>
        <td>Integer array</td>
        <td>Array containing integers <code>1</code>, <code>2</code> and <code>3</code></td>
    </tr>
    <tr>
        <td><code>double_array_a</code></td>
        <td>&mdash;</td>
        <td>Double array</td>
        <td>Array containing doubles <code>-1.1</code>, <code>-2.2</code> and <code>-3.3</code></td>
    </tr>
    <tr>
        <td><code>string_array_a</code></td>
        <td>&mdash;</td>
        <td>Double array</td>
        <td>Array containing strings <code>'a'</code>, <code>'b'</code> and <code>'c'</code></td>
    </tr>
    <tr>
        <td><code>node_b</code></td>
        <td><code>integer_a</code></td>
        <td>&mdash;</td>
        <td>Integer</td>
        <td><code>0</code></td>
    </tr>
</tbody></table>


## 4.2&emsp;Where to Place

The YAML file is typically stored in a directory `params`
or `config`. This directory is in the same package as the nodes where the parameters should be loaded.

The following example shows where the `params` directory should go under a package.

<table><tbody><tr><td>
    <details open>
    <summary><code>workspace_a/</code>&emsp;Workspace.</summary>
    <dl>
        <dd><details open> 
            <summary><code>src/</code></summary>
            <dl>
                <dd><details open>
                    <summary><code>pkg_a/</code>&emsp;Python package.</summary>
                    <dl>
                        <dd><details open>
                            <summary><code>params</code>&emsp; The folder containing the YAML file.</summary>
                            <dl>
                                <dd><code>params_a.yaml</code>&emsp; The YAML file.</dd>
                            </dl>
                        </details></dd>
                        <dd><details>
                            <summary><code>pkg_a/</code></summary>
                            <dl>
                                <dd><code>__init__.py</code></dd>
                                <dd><code>node_a.py</code>&emsp;A node.</dd>
                                <dd><code>node_b.py</code>&emsp;A node.</dd>
                            </dl>
                        </details></dd>
                        <dd>...</dd>
                    </dl>
                </details></dd>
            </dl>
        </details></dd>
    </dl>
    </details>
</td></tr></tbody></table>

## 4.3&emsp;Installing with `setup.py`

The YAML file needs to be available in the installation directory after building, so a launch file can find it.  The following code **should only be inserted once** to install the `params` directory and its contents.

In `setup.py`, 
1. Import `glob`.
2. Look for the `data_files=` keyword argument, and add the following line. The example assumes that the `params` directory is used:
    ```python
    from setuptools import find_packages, setup
    from glob import glob

    package_name = 'pkg_a' 

    setup(
        #...
        data_files=[
            #...
            ('share/' + package_name + '/params', glob('params/*.yaml')),
        ]
        #...
    )
    ```
3. Build with `colcon build --symlink-install`. Unless the YAML file's  name changes, or a new YAML file is created, re-building is not necessary.

## 4.4&emsp;Loading with `ros2 run`

When a YAML file containing parameters from multiple nodes is loaded into a node, **only the parameters from the same node are loaded**.
For example, if the YAML file from the previous section is loaded into the node `node_b`, ROS2 looks for all entries under `node_b` in the YAML file, and ignores those under `namespace_a/node_a`.
As such, only the `integer_a` entry under `node_b` is loaded.

Suppose that the file `params_a.yaml` is stored in the `params` directory in the package `pkg_a`. Let the workplace be `workspace_a`.
To load it with the `ros2 run` CLI (make sure to source first),
```bash
ros2 run pkg_a node_a --ros-args --param-file workspace_a/src/pkg_a/params/params_a.yaml
```
Of course, the YAML file path is relative to the directory where the terminal is currently in.

## 4.5&emsp;Loading with Launch File
To use the parameter file in the launch file, make sure to follow the steps in [4.3 Installing With `setup.py`](#43installing-with-setuppy).

To load into a node in the launch file, make use of the `parameters=` keyword argument in `Node`. The example follows the same names as previous examples.
```python
# ...
pkg_a = FindPackageShare('pkg_a')
# ...
node_node_a = Node(
    # ...
    parameters=[PathJoinSubstitution([pkg_a, 'params', 'params_a.yaml'])],
    # ...
)
# ...
```

## 4.6&emsp;Tasks

Design a YAML file `prms.yaml`, placed in the `params` directory of the `rb2301_tutorial` package. The file must contain the following parameter values for the `parameters` node created in the previous section:

| Parameter | Value |
|-|-|
| `values.bool_v` | `True` if last numeric digit is even, `False` if odd. (e.g. `True` for `456X`) |
| `values.int_v` | Last numeric digit (e.g. `6` for `456X`). |
| `values.dbl_v` | Last two numeric digits (e.g. `56` for `456X`). |
| `values.str_v` | Your last name / surname (e.g. `Doe` for `John Doe`). |
| `bool_arr_v` | A two-element array based on the last two numeric digits. If the digit is even, then the corresponding entry is `True`, otherwise it is `False` (e.g. `[False, True]` for `456X`). |
| `int_arr_v` | A three-element array based on the last three numeric digits (e.g. `[4,5,6]` for `456X`.) |
| `dbl_arr_v` | A three-element array based on the last three numeric digits, divided by 2 (e.g. `[2.0, 2.5, 3.0]` for `456X`) | 
| `str_arr_v` | An array that is at least one-element long containing your name, with the number of elements corresponding to the number of words in your name (e.g. `['John', 'Doe']` for `John Doe`). |

**[Q2a]** What are the last four characters of your matric number? (e.g. `456X` from `A0123456X`).

**[Q2b]** What are the new line(s) to add to the `setup.py` package so that the `params` folder and all of its contents are installed? Remember to build the workspace if the YAML file is created for the first time.

**[Q2c]** What is the `ros2 run` CLI to run the node `parameters`, created in the previous section, with this parameter file? Assume that the terminal is already in the workspace.

**[Q2d]** Cut and paste the entire contents of the YAML file.