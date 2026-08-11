9b&emsp;Parameter Services (Other Nodes)
==============

***RB2301 Robot Programming***

**&copy; Lai Yan Kai, National University of Singapore**

The previous chapter focuses on modifying parameters from the same node. 
To be able to modify **parameters from other nodes**, **parameter services** must be used. 
Parameter services are essentially ROS2 services, and the same CLI from [03_CLI.md](03_CLI.md) and code in [05_Node.md](05_Node.md#24create-a-basic-service-server) can be used to interact with these services.

# Table of Contents

[1&emsp;Parameter Services](#1parameter-services)

&emsp;[1.1&emsp;Automatically Populated Services](#11automatically-populated-services)

&emsp;[1.2&emsp;Supporting Message Definitions](#12supporting-message-definitions)

&emsp;[1.3&emsp;Get Parameter Example](#13get-parameter-example)

&emsp;[1.4&emsp;Set Parameter Example](#14set-parameter-example)

[2&emsp;Tasks](#2tasks)

[Appendix A&emsp;Parameter Service Interfaces](#appendix-aparameter-service-interfaces)

&emsp;[A.1&emsp;`DescribeParameters`](#a1describeparameters)

&emsp;[A.2&emsp;`GetParameters`](#a2getparameters)

&emsp;[A.3&emsp;`GetParameterTypes`](#a3getparametertypes)

&emsp;[A.4&emsp;`ListParameters`](#a4listparameters)

&emsp;[A.5&emsp;`SetParameters`](#a5setparameters)

&emsp;[A.6&emsp;`SetParametersAtomically`](#a6setparametersatomically)

[Appendix B&emsp;Parameter Message Definitions](#appendix-bparameter-message-definitions)

&emsp;[B.1&emsp;FloatingPointRange](#b1floatingpointrange)

&emsp;[B.2&emsp;IntegerRange](#b2integerrange)

&emsp;[B.3&emsp;ListParametersResult](#b3listparametersresult)

&emsp;[B.4&emsp;Parameter](#b4parameter)

&emsp;[B.5&emsp;ParameterDescriptor](#b5parameterdescriptor)

&emsp;[B.6&emsp;ParameterType](#b6parametertype)

&emsp;[B.7&emsp;ParameterValue](#b7parametervalue)

&emsp;[B.8&emsp;SetParametersResult](#b8setparametersresult)

# 1&emsp;Parameter Services

## 1.1&emsp;Automatically Populated Services
Parameters are read or written using services. Even the `ros2 param` CLI uses these services to modify parameters.
As such, all nodes are **automatically populated** by the services in the following table.
Each service has its own unique service interface (see [Appendix A Parameter Service Interfaces](#appendix-aparameter-service-interfaces)). 
Suppose the node is called `node_a`. Then, the services are:

|| Service | Interface Type and Description |
|-|-|-|
|1.| `/node_a/describe_parameters` |`rcl_interfaces/srv/DescribeParameters`. Returns descriptions about the requested parameters &mdash; their name, type, help text, and constraints. |
|2.| `/node_a/get_parameters` |`rcl_interfaces/srv/GetParameters`. Returns the parameter value and their descriptions. |
|3.| `/node_a/get_parameter_types` |`rcl_interfaces/srv/GetParameterTypes`. Returns the types of the requested parameters. |
|4.| `/node_a/list_parameters` |`rcl_interfaces/srv/ListParameters`. Lists the parameters in a node, filtered based on the requested prefixes and depth. |
|5.| `/node_a/set_parameters` |`rcl_interfaces/srv/SetParameters`. Sets the requested parameters. |
|6.| `/node_a/set_parameters_atomically` |`rcl_interfaces/srv/SetParametersAtomically`. Sets all parameters in one shot, instead of triggering multiple callbacks to set the parameters. If any parameter setting fails, all parameters will not be set. This prevents the node from operating in a partially updated state that can cause unexpected behaviors. |


## 1.2&emsp;Supporting Message Definitions
Most of the service interfaces rely on message definitions (see [Appendix B Parameter Message Definitions](#appendix-bparameter-message-definitions)) from the same `rcl_interfaces` package:

| | Message Type | Description |
|-|-|-|
|1.| `rcl_interfaces/msg/FloatingPointRange` | Specifies a floating point constraint for numeric parameters. |
|2.| `rcl_interfaces/msg/IntegerRange` | Specifies an integer constraint for numeric parameters. | 
|3.| `rcl_interfaces/msg/ListParametersResult` | A response by the `list_parameters` service. |
|4.| `rcl_interfaces/msg/Parameter` | Specifies a parameter's name and value. |
|5.| `rcl_interfaces/msg/ParameterDescriptor` | Describes a parameter's name, type, constraints, and its purpose. |
|6.| `rcl_interfaces/msg/ParameterType` | Not a message, but an enum containing integer values representing the various parameter types. These integer values are used by various services to determine the parameter types. |
|7.| `rcl_interfaces/msg/ParameterValue` | Contains a parameter's value in the data field corresponding to the parameter's type. This definition allows for dynamic typing. |
|8.| `rcl_interfaces/msg/SetParametersResult` | A response by the `set_parameters` service. |

## 1.3&emsp;Get Parameter Example
The following example shows how to get the values of parameters `prm_a` and `prm_b` from another node `node_b` in a timer callback.
Use `ros2 interface show` or VSCode Intellisense to examine the fields within the `rcl_interfaces` package's interfaces.

```python
import rclpy
from rclpy.node import Node
from rcl_interfaces.srv import GetParameters

class NodeA(Node):
    get_params_future = None

    def __init__(self):
        super().__init__('node_a')

        self.get_params_cli = self.create_client(GetParameters, '/node_b/get_parameters')

        self.timer = self.create_timer(0.5, self.timer_callback)

    def timer_callback(self):
        if self.get_params_future is None:
            request = GetParameters.Request()
            request.names = ['prm_a', 'prm_b']
            self.get_params_future = self.get_params_cli.call_async(request)
            print(f'Send Request')

        elif self.get_params_future.done():
            response = self.get_params_future.result()

            print(f'Got Response: {response}')
            print(f'\tGot prm_a: {response.values[0].integer_value}') # if it prm_a is integer type
            print(f'\tGot prm_b: {response.values[1].double_array_value}') # if prm_b is double array type

            self.get_params_future = None # reset the future so another request can be sent.

def main(args=None):
    rclpy.init(args=args)
    node = NodeA()
    rclpy.spin(node) 
    rclpy.shutdown()
if __name__ == '__main__':
    main()
```

## 1.4&emsp;Set Parameter Example
The following example shows how to set parameters `prm_a` and `prm_b` from another node `node_b`. 
Use `ros2 interface show` or VSCode Intellisense to examine the fields within the `rcl_interfaces` interfaces.

```python
import rclpy
from rclpy.node import Node
from rcl_interfaces.msg import ParameterType, Parameter
from rcl_interfaces.srv import SetParameters

class NodeA(Node):
    set_params_future = None

    def __init__(self):
        super().__init__('node_a')

        self.get_params_cli = self.create_client(SetParameters, '/node_b/set_parameters')

        self.timer = self.create_timer(0.5, self.timer_callback)

    def timer_callback(self):
        if self.set_params_future is None:
            request = SetParameters.Request()

            new_prm_a = Parameter()
            new_prm_a.name = 'prm_a'
            new_prm_a.value.type = ParameterType.PARAMETER_INTEGER # if integer
            new_prm_a.value.integer_value = 255

            new_prm_b = Parameter()
            new_prm_b.name = 'prm_b'
            new_prm_b.value.type = ParameterType.PARAMETER_DOUBLE_ARRAY # if double array
            new_prm_b.value.double_array_value = [1.1, 2.2, 3.3]

            request.parameters = [new_prm_a, new_prm_b]

            print(f'Sending Request')
            
            self.set_params_future = self.set_params_cli.call_async(request)

        elif self.set_params_future.done():
            response = self.set_params_future.result()

            print(f'Got Response: {response}')

            self.set_params_future = None # reset service

def main(args=None):
    rclpy.init(args=args)
    node = NodeA()
    rclpy.spin(node) 
    rclpy.shutdown()
if __name__ == '__main__':
    main()
```

# 2&emsp;Tasks


1. Run `turtlesim` and list all the available services. Then try to get a parameter using the CLI.

    **[Q1a]** What is the `ros2 param` CLI to get the value of the `background_b` parameter?

    **[Q1b]** State the full name for `turtlesim`'s get-parameter service.

    **[Q1c]** State the service interface type of the get-parameter service.

    **[Q1d]** Copy the output of `ros interface show` for the get-parameter service's interface type.

    **[Q1e]** What is the `ros2 service` CLI to get the value of the `background_b` and `background_r` parameters? Hint: use `[]` to specify arrays.

2. Now, try to set a parameter using the CLI.

    **[Q1f]** What is the `ros2 param` CLI to set the value of `background_r` to `200`?

    **[Q1g]** State the full name for `turtlesim`'s set-parameter service.

    **[Q1h]** State the service interface type of the set-parameter service.

    **[Q1i]** Copy the output of `ros interface show` for the set-parameter service's interface type.

    **[Q1j]** What is the `ros2 service` CLI to set the value of `background_g` to `100` and `background_b` to `50`? Hint: only the `name`, `type` (integer specified in [B6 ParameterType](#b6parametertype)), and `integer_value` fields are required. Fields may not belong to the same class.

3. Design a node called `parameter_services` that can be run with `ros2 run rb2301_tutorial prm_srvs`.

4. Implement the service clients in the node constructor that can send requests to `turtlesim`'s get-parameter and set-parameter services.

    **[Q1k]** What are the lines of code to implement the service clients?

5. Implement a timer callback that runs every 0.5 seconds.

    1. Implement a service request call to obtain the values for `turtlesim`'s `background_r` and `background_g` values.

    2. Once the values are obtained (which may not occur in the current call), send another request that increments `background_r`'s value by `50` and `background_g`'s value by `-40`. 
    The new values should stay within 0 and 255 by using the modulo `%` operator.

    3. Print the new values by replacing `new_r` and `new_g` in the print statement below:

        ```python
        print(f'Setting Red({new_r:3d}) and Green({new_g:3d})')    
        ```

    **[Q1l]** What is the code for the timer callback? Include the function declaration and the body of the callback. For example:
        
        ```python
        def timer_callback(self):
            if self.get_params_future is None:
                # ...
            if self.get_params_future.done():
                # ...
            if self.set_params_future is None:
                # ...
            if self.set_params_future.done():
                # ...
        ```

# Appendix A&emsp;Parameter Service Interfaces
Every parameter service has its own service interface definitions. The service interfaces can be found in the `rcl_interfaces` package. 

## A.1&emsp;`DescribeParameters`
The full type is `rcl_interfaces/srv/DescribeParameters`.
<dl>
    <dd>
        <details open>
            <summary><b>Request</b></summary>
            <ul>
                <li>
                    <code>string[]</code>&emsp;<code>name</code>&emsp;A string array of parameter names for the requested parameters.
                </li>
            </ul>
        </details>
    </dd>
    <dd>
        <details open>
            <summary><b>Response</b></summary>
            <ul>
                <li>
                    <code>ParameterDescriptor[]</code>&emsp;<code>descriptors</code>&emsp;An array of <code>ParameterDescriptor</code> elements describing the requested parameters in the same order as the request.
                </li>
            </ul>
        </details>
    </dd>
</dl>

## A.2&emsp;`GetParameters`
The full type is `rcl_interfaces/srv/GetParameters`.
<dl>
    <dd>
        <details open>
            <summary><b>Request</b></summary>
            <ul>
                <li>
                    <code>string[]</code>&emsp;<code>name</code>&emsp;A string array of parameter names for the requested parameters.
                </li>
            </ul>
        </details>
    </dd>
    <dd>
        <details open>
            <summary><b>Response</b></summary>
            <ul>
                <li>
                    <code>ParameterValue[]</code>&emsp;<code>values</code>&emsp;An array of <code>ParameterValue</code> objects that provides information about each requested parameter, in the same order as the request.
                </li>
            </ul>
        </details>
    </dd>
</dl>

## A.3&emsp;`GetParameterTypes`
The full type is `rcl_interfaces/srv/GetParameterTypes`.
<dl>
    <dd>
        <details open>
            <summary><b>Request</b></summary>
            <ul>
                <li>
                    <code>string[]</code>&emsp;<code>name</code>&emsp;A string array of parameter names for the requested parameters.
                </li>
            </ul>
        </details>
    </dd>
    <dd>
        <details open>
            <summary><b>Response</b></summary>
            <ul>
                <li>
                    <code>uint8[]</code>&emsp;<code>types</code>&emsp;The array of integers describing the types of the requested parameters, in the same order as the request. The type is enumerated by the enum <code>ParameterType</code>.
                </li>
            </ul>
        </details>
    </dd>
</dl>

## A.4&emsp;`ListParameters`
The full type is `rcl_interfaces/srv/ListParameters`.
<dl>
    <dd>
        <details open>
            <summary><b>Request</b></summary>
            <ul>
                <li>
                    <code>uint64</code>&emsp;<code>depth</code>&emsp;If 0, gets all parameters recursively with unlimited depth. Otherwise, gets up to the depth defined here.
                </li>
                <li>
                    <code>string[]</code>&emsp;<code>prefixes</code>&emsp;Returns all parameters that are grouped under a prefix listed in here.
                </li>
            </ul>
        </details>
    </dd>
    <dd>
        <details open>
            <summary><b>Response</b></summary>
            <ul>
                <li>
                    <code>ListParametersResult[]</code>&emsp;<code>result</code>&emsp;An array of <code>ListParametersResult</code> objects in the same order as the request and up to the specified depth. The names of the parameters and related  (deepest) prefixes are indicated. 
                </li>
            </ul>
        </details>
    </dd>
</dl>

## A.5&emsp;`SetParameters`
The full type is `rcl_interfaces/srv/SetParameters`.
<dl>
    <dd>
        <details open>
            <summary><b>Request</b></summary>
            <ul>
                <li>
                    <code>Parameter[]</code>&emsp;<code>parameters</code>&emsp;An array of <code>Parameter</code> objects. The name and value of each parameter to set is set in each object.
                </li>
            </ul>
        </details>
    </dd>
    <dd>
        <details open>
            <summary><b>Response</b></summary>
            <ul>
                <li>
                    <code>SetParametersResult[]</code>&emsp;<code>result</code>&emsp;An array of <code>SetParametersResult</code> objects in the same order as the request. Each object has a boolean indicating if the setting is successful or not, and a string indicating the reason if not successful.
                </li>
            </ul>
        </details>
    </dd>
</dl>

## A.6&emsp;`SetParametersAtomically`
Has the same contents as `rcl_interfaces/srv/SetParameters`.

# Appendix B&emsp;Parameter Message Definitions
The service interface definitions have properties (fields) implemented as message definitions. The message definitions can be seen as building blocks for the service interfaces, the service interfaces as building blocks for the services, and the services as building blocks for parameter setting and getting.

## B.1&emsp;FloatingPointRange
The full message type is `rcl_interfaces/msg/FloatingPointRange`. Provides information about the valid values that a numeric parameter can hold.

Comparisons may be susceptible to floating point errors. It is probably better to implement the validation in your own code.

<ul>
    <li><code>float64</code>&emsp;<code>from_value</code>&emsp;Smallest value, inclusive.</li>
    <li><code>float64</code>&emsp;<code>to_value</code>&emsp;Largest value, inclusive.</li>
    <li><code>float64</code>&emsp;<code>step</code>&emsp;A positive number. Valid numbers for this parameter must be a multiples of <code>step</code> starting from <code>from_value</code>. <code>to_value</code> is always a valid number even if it is not multiples of <code>step</code> from <code>from_value</code>.</li>
</ul>

## B.2&emsp;IntegerRange
The full message type is `rcl_interfaces/msg/IntegerRange`. Provides information about the valid values that a numeric parameter can hold.

<ul>
    <li><code>int64</code>&emsp;<code>from_value</code>&emsp;Smallest value, inclusive.</li>
    <li><code>int64</code>&emsp;<code>to_value</code>&emsp;Largest value, inclusive.</li>
    <li><code>uint64</code>&emsp;<code>step</code>&emsp;A positive number. Valid numbers for this parameter must be a multiples of <code>step</code> starting from <code>from_value</code>. <code>to_value</code> is always a valid number even if it is not multiples of <code>step</code> from <code>from_value</code>.</li>
</ul>

## B.3&emsp;ListParametersResult
The full message type is `rcl_interfaces/msg/ListParametersResult`. Used primarily as a response by the `list_parameters` service.
<ul>
    <li><code>string[]</code>&emsp;<code>names</code>&emsp;An string array containing the names of the parameters that satisfy the request.</li>
    <li><code>string[]</code>&emsp;<code>prefixes</code>&emsp;A string array containing the prefix of the parameters that satisfy the request.</li>
</ul>

## B.4&emsp;Parameter
The full message type is `rcl_interfaces/msg/Parameter`. Contains a parameter's name and value.

<ul>
    <li><code>string</code>&emsp;<code>name</code>&emsp;Name of the parameter.</li>
    <li><code>ParameterValue</code>&emsp;<code>value</code>&emsp;The value of the parameter.</li>
</ul>

## B.5&emsp;ParameterDescriptor
The full message type is `rcl_interfaces/msg/ParameterDescriptor`. Provides help text and constraint information about a parameter.

<ul>
    <li><code>string</code>&emsp;<code>name</code>&emsp;Name of the parameter.</li>
    <li><code>uint8</code>&emsp;<code>type</code>&emsp;Type of the parameter. The number can be found in <code>ParameterType</code>.</li>
    <li><code>string</code>&emsp;<code>description</code>&emsp;Help text for the parameter.</li>
    <li><code>string</code>&emsp;<code>additional_constraints</code>&emsp;Help text about the value constraints.</li>
    <li><code>bool</code>&emsp;<code>read_only=False</code>&emsp;If <code>True</code>, the value cannot be changed after initialization.</li>
    <li><code>FloatingPointRange</code>&emsp;<code>floating_point_range</code>Specifies the numeric constraint using floating point values. Should not be used with <code>integer_range</code>.</li>
    <li><code>IntegerRange</code>&emsp;<code>integer_range</code>&emsp;Specifies the numeric constraint using integer values. Should not be used with <code>floating_range</code>.</li>
</ul>

## B.6&emsp;ParameterType
The full message type is `rcl_interfaces/msg/ParameterType`. Used primarily as an enum for the different parameter types. It is not used for sending messages.

<ul>
    <li><code>uint8</code>&emsp;<code>PARAMETER_NOT_SET=0</code>&emsp;</li>
    <li><code>uint8</code>&emsp;<code>PARAMETER_BOOL=1</code>&emsp;</li>
    <li><code>uint8</code>&emsp;<code>PARAMETER_INTEGER=2</code>&emsp;</li>
    <li><code>uint8</code>&emsp;<code>PARAMETER_DOUBLE=3</code>&emsp;</li>
    <li><code>uint8</code>&emsp;<code>PARAMETER_STRING=4</code>&emsp;</li>
    <li><code>uint8</code>&emsp;<code>PARAMETER_BYTE_ARRAY=5</code>&emsp;</li>
    <li><code>uint8</code>&emsp;<code>PARAMETER_BOOL_ARRAY=6</code>&emsp;</li>
    <li><code>uint8</code>&emsp;<code>PARAMETER_INTEGER_ARRAY=7</code>&emsp;</li>
    <li><code>uint8</code>&emsp;<code>PARAMETER_DOUBLE_ARRAY=8</code>&emsp;</li>
    <li><code>uint8</code>&emsp;<code>PARAMETER_STRING_ARRAY=9</code>&emsp;</li>
</ul>

## B.7&emsp;ParameterValue
The full message type is `rcl_interfaces/msg/ParameterValue`. The `type` field indicates which `_value` field will be used to store the parameter value. `_value` fields that do not correspond to the `type` field are not used.

<ul>
    <li><code>uint8</code>&emsp;<code>type</code>&emsp;Use the integer value that corresponds to the <code>ParameterType</code> enum.</li>
    <li><code>bool</code>&emsp;<code>bool_value</code></li>
    <li><code>int64</code>&emsp;<code>integer_value</code></li>
    <li><code>float64</code>&emsp;<code>double_value</code></li>
    <li><code>string</code>&emsp;<code>string_value</code></li>
    <li><code>byte[]</code>&emsp;<code>byte_array_value</code></li>
    <li><code>bool[]</code>&emsp;<code>bool_array_value</code></li>
    <li><code>int64[]</code>&emsp;<code>integer_array_value</code></li>
    <li><code>float64[]</code>&emsp;<code>double_array_value</code></li>
    <li><code>string[]</code>&emsp;<code>string_array_value</code></li>
</ul>

## B.8&emsp;SetParametersResult
The full message type is `rcl_interfaces/msg/SetParametersResult`. This message is used primarily as the response for the `set_parameters` service.

<ul>
    <li><code>bool</code>&emsp;<code>successful</code>&emsp;A boolean indicating if the setting is successful.</li>
    <li><code>string</code>&emsp;<code>reason</code>&emsp; The reason for failure, if not successful.</li>
</ul>