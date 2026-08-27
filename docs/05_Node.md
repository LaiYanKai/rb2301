05&emsp;Coding a Basic ROS2 Node
==============

***RB2301 Robot Programming***

**&copy; Lai Yan Kai, National University of Singapore**

The basic Pythonic structure of a ROS2 node is provided. Use the structure to begin coding the Python scripts for the ROS2 nodes that were created in the previous chapter.

# Table of Contents
[1&emsp;Design Concepts](#1design-concepts)

&emsp;[1.1&emsp;ROS2 Concepts](#11ros2-concepts)

&emsp;[1.2&emsp; Concurrency Concepts](#12-concurrency-concepts)

[2&emsp;The Basic Pythonic structure](#2the-basic-pythonic-structure)

&emsp;[2.1&emsp;Create a Timer](#21create-a-timer)

&emsp;[2.2&emsp;Create a Basic Topic Subscriber](#22create-a-basic-topic-subscriber)

&emsp;[2.3&emsp;Create a Basic Topic Publisher](#23create-a-basic-topic-publisher)

&emsp;[2.4&emsp;Create a Basic Service Server](#24create-a-basic-service-server)

&emsp;[2.5&emsp;Create a Basic Service Client](#25create-a-basic-service-client)

&emsp;[2.6&emsp;Read Parameters](#26read-parameters)

[3&emsp;Tasks](#3tasks)

# 1&emsp;Design Concepts

## 1.1&emsp;ROS2 Concepts
The following lists the core design concepts of designing a node:
| Concept | Purpose  | 
| - | - | 
| **Timer** | Calls a callback function at a fixed frequency. The callback function performs a step in a task that depends on time, such as controller and localization tasks. |
| **Subscriber** | Calls a callback everytime a message is received from a subscribed topic. The callback function typically transfers the message from the topic to the node so that other functions can use the message. |
| **Publisher** | Publishes a message directly into a topic. |
| **Service Client** | Performs a request to a topic. The client can wait for the response asynchronously or synchronously. A callback can be assigned for any received response (see later chapters). | 
| **Service Server** | Calls a callback function everytime a service request is received. The callback function processes the request and returns a response. |
| **Read Parameter** | Gets a ROS2 parameter. Only parameters from the same node can only be declared and read. See later chapters for more functionality. | 

Every node can have **any number** of the above design concepts, where there can be multiple subscribers, publishers, clients, and service servers functioning at once.

## 1.2&emsp; Concurrency Concepts

ROS2 borrows a lot from concurrent programming, because a typical robotic system has many simultaneously interacting components.

The table below lists the major concurrency concepts in ROS2. For this course, we focus only on the concept of callbacks. After reading the table, please keep in mind that:
- **All callbacks cannot be run concurrently**. Since we use the single threaded executor, callbacks can only be run one after another. 
- **All callbacks should return quickly**. Unless the callback handles long-running tasks like service request callbacks, every callback should complete as fast as possible so that other callbacks can run. This is in line with conventional best practice.

| Concept | Description |
| - | - | 
| **Callbacks** | A callback is a function that is called every time an event is triggered. Such an event includes when a message is received (subscriber), a service request is received (service server), or when some time has passed since the last call (timer). A callback should complete quickly so that other callbacks can run. |
| **Executors** | **[Not required by this course]**. Works with the callback groups to coordinate when to call the callbacks. In this course, we use the default, which is the single-threaded executor. This means that all callbacks cannot be run concurrently and only one after another. The other executor is the multi-threaded executor. | 
| **Callback Groups** | **[Not required by this course]**. Can only be used with the multi-threaded executor. Callbacks in the same group cannot be run concurrently. Callbacks from different groups can run concurrently with each other. |
| **Spin** | **[Not required by this course]**. Queries the middleware for new events that will call other callbacks. Spinning is done automatically between callbacks, and can be done in a callback by using `spin_once()`. Whether the callbacks for these new events are triggered depend on the executor and callback groups. | 


# 2&emsp;The Basic Pythonic structure
In this section Python templates for the ROS2 concepts are provided. 
These templates are elementary, but covers the majority of use cases. 
Advanced concepts such as blocking and non-blocking calls are not covered, and will be provided in the later, more advanced chapters. 

As such, use this section as a primer to begin coding in ROS2 Python, and once you are familiar with the content, question what is lacking about these basic implementations.

The base template of a Python ROS2 Node is provided below. There are five locations where code needs to be filled:
```python
# 1. IMPORT
import rclpy
from rclpy.node import Node

# NODE CLASS
class SomeROSNode(Node):
    def __init__(self):
        super().__init__('some_ros_node')
        # 2. NODE PROPERTIES

        # 3. NODE HANDLES

    # 4. NODE CALLBACKS

    # 5. HOW TO USE


# MAIN BOILER PLATE
def main(args=None):
    rclpy.init(args=args)
    node = SomeROSNode() # same name as the class above.
    rclpy.spin(node)
    rclpy.shutdown()
if __name__ == '__main__':
    main()
```

| | Where | Purpose |
|-|-|-|
| 1. | **Import** | Import statements, to be appended after the first two import lines. Any libraries that are imported must be included in `package.xml`. |
| 2. | **Node Properties** | The node is implemented as a class derived from the ROS2 `Node` class. Initialize variables at this location if the variables has to be read and written across different callbacks. |
| 3. | **Node Handles** | Initializes handles (variables) to timers, subscribers, publishers, service clients, service servers, etc. |
| 4. | **Node Callbacks** | Place callback functions here. Callback functions are class methods, like the `__init__` constructor. |
| 5. | **How to Use** | Contains example code to work with the handles. |


## 2.1&emsp;Create a Timer
A timer runs a callback at fixed intervals of time. It is commonly used by nodes to run a critical function at a fixed frequency, such as controller and localization nodes.

Suppose we want to run a callback every `duration` seconds:
The following beginner Python syntax is suggested:
<table><tbody>
    <tr>
        <th></th>
        <th>Append To</th>
        <th>Code</th>
    </tr>
    <tr>
        <td>1.</td>
        <td>Import</td>
        <td>&mdash;</td>
    </tr>
    <tr>
        <td>2.</td>
        <td>Node Properties</td>
        <td>&mdash;</td>
    </tr>
    <tr>
        <td>3.</td>
        <td>Node Handles</td>
        <td>
            Initialize the handle to the subscriber.
            <ul>
                <li><code>duration</code>: The time interval in seconds to run the callback. If an interval is missed, the callback will not be called for the interval.</li>
                <li><code>self.timer_callback</code>: The callback to at every interval.</li>
            </ul>
            <pre lang="python">self.timer = self.create_timer(duration, self.timer_callback)</pre></td>
    </tr>
    <tr>
        <td>4.</td>
        <td>Node Callbacks</td>
        <td>This callback is run once every interval. It should return quickly so that other callbacks can be called.
<pre lang="python">
def timer_callback(self):
    # do something
</pre>
        </td>
    </tr>
    <tr>
        <td>5.</td>
        <td>How to Use</td>
        <td>&mdash;</td>
    </tr>
</tbody></table>

Suppose we want to run a loop every 0.5 seconds and print out the number of seconds and nanoseconds on the clock:

```python
# 1. IMPORT
import rclpy
from rclpy.node import Node

# NODE CLASS
class TimingNode(Node):

    def __init__(self):
        super().__init__('timing_node')
        # 2. NODE PROPERTIES

        # 3. NODE HANDLES
        self.timer = self.create_timer(0.5, self.timer_callback)


    # 4. NODE CALLBACKS
    def timer_callback(self):
        time_now = self.get_clock().now().seconds_nanoseconds()
        print(f'sec: {time_now[0]}, nanosec: {time_now[1]}')

    # 5. HOW TO USE        

# MAIN BOILER PLATE
def main(args=None):
    rclpy.init(args=args)
    node = TimingNode() # this changed
    rclpy.spin(node)
    rclpy.shutdown()
if __name__ == '__main__':
    main()
```

## 2.2&emsp;Create a Basic Topic Subscriber

Suppose we want to subscribe to a `lib_a/msg/MsgTypeA` message to the topic `/topic_a`.
The following beginner Python syntax is suggested:
<table><tbody>
    <tr>
        <th></th>
        <th>Append To</th>
        <th>Code</th>
    </tr>
    <tr>
        <td>1.</td>
        <td>Import</td>
        <td>
        Import the message class.
        <pre lang="python">from lib_a.msg import MsgTypeA</pre>
        In <code>package.xml</code>, add the <code>&lt;depend&gt;</code> tag for the library.
<pre lang="xml">
&lt;depend&gt;lib_a&lt;/depend&gt;
</pre>
        </td>
    </tr>
    <tr>
        <td>2.</td>
        <td>Node Properties</td>
        <td>Store a copy of the message for use by the other class methods. Initially nothing.
        <pre lang="python">topic_a_msg = None</pre>
        </td>
    </tr>
    <tr>
        <td>3.</td>
        <td>Node Handles</td>
        <td>
            Initialize the handle to the subscriber.
            <ul>
                <li><code>MsgTypeA</code>: The imported message class. </li>
                <li><code>'/topic_a'</code>: The string of the topic name. </li>
                <li><code>self.topic_a_sub_callback</code>: The callback method below. The callback is called once on each message that arrives from the topic.</li>
                <li><code>10</code>: The queue size, which is the maximum number of past messages to store by the subscriber if the computer is too busy to call the subscriber callbacks (depends on QoS settings).</li>
            </ul>
            <pre lang="python">self.topic_a_sub = self.create_subscription(MsgTypeA, '/topic_a', self.topic_a_sub_callback, 10)</pre></td>
    </tr>
    <tr>
        <td>4.</td>
        <td>Node Callbacks</td>
        <td>In usual circumstances, the callback method is <b>very fast</b>. The whole or parts of the arrived message <code>msg</code> is copied into the class for use by other methods.
<pre lang="python">
def topic_a_sub_callback(self, msg):
    self.topic_a_msg = msg
</pre>
        </td>
    </tr>
    <tr>
        <td>5.</td>
        <td>How to Use</td>
        <td>
            Use the copied message directly.
            <pre lang="python"># use self.topic_a_msg</pre>
        </td>
    </tr>
</tbody></table>

For example, suppose a node called `Controller` listens only to a topic `/odom` with the message type `nav_msgs/msg/Odometry`. 
Suppose the x-coordinate in the message has to be printed (see https://docs.ros.org/en/jazzy/p/nav_msgs/ and https://docs.ros.org/en/jazzy/p/geometry_msgs/).

```python
# 1. IMPORT
import rclpy
from rclpy.node import Node
from nav_msgs.msg import Odometry

# NODE CLASS
class Controller(Node):

    def __init__(self):
        super().__init__('controller')
        # 2. NODE PROPERTIES
        self.odom_msg = None

        # 3. NODE HANDLES
        self.odom_sub = self.create_subscription(Odometry, '/odom', self.odom_sub_callback, 10)

    # 4. NODE CALLBACKS
    def odom_sub_callback(self, msg):
        self.odom_msg = msg
        
    def some_function(self):
        # 5. HOW TO USE
        print(self.odom_msg.pose.pose.position.x)

# MAIN BOILER PLATE
def main(args=None):
    rclpy.init(args=args)
    node = Controller() # this changed
    rclpy.spin(node)
    rclpy.shutdown()
if __name__ == '__main__':
    main()
```

Then, add the `nav_msgs` library to `package.xml`.

## 2.3&emsp;Create a Basic Topic Publisher
Suppose we want to publish a `lib_a/msg/MsgTypeA` message to the topic `/topic_a`.
The following beginner Python syntax is suggested:
<table><tbody>
    <tr>
        <th></th>
        <th>Append To</th>
        <th>Code</th>
    </tr>
    <tr>
        <td>1.</td>
        <td>Import</td>
        <td>
        Import the message class.
        <pre lang="python">from lib_a.msg import MsgTypeA</pre>
        In <code>package.xml</code>, add the <code>&lt;depend&gt;</code> tag for the library.
<pre lang="xml">
&lt;depend&gt;lib_a&lt;/depend&gt;
</pre>
        </td>
    </tr>
    <tr>
        <td>2.</td>
        <td>Node Properties</td>
        <td> &mdash;
        </td>
    </tr>
    <tr>
        <td>3.</td>
        <td>Node Handles</td>
        <td>
            Initialize the handle to the publisher.
            <ul>
                <li><code>MsgTypeA</code>: The imported message class. </li>
                <li><code>'/topic_a'</code>: The string of the topic name. </li>
                <li><code>10</code>: The queue size, which is the maximum number of past messages to store if subscribers were not yet active and are unable to receive messages (depends on QoS settings). </li>
            </ul>
            <pre lang="python">self.topic_a_pub = self.create_publisher(MsgTypeA, '/topic_a', 10)</pre></td>
    </tr>
    <tr>
        <td>4.</td>
        <td>Node Callbacks</td>
        <td>&mdash;</td>
    </tr>
    <tr>
        <td>5.</td>
        <td>How to Use</td>
        <td>
            Create a message, fill up the message, and then use the handle to publish the message.
<pre lang="python">
topic_a_msg = MsgTypeA()
# ... fill message contents...
self.topic_a_pub.publish(topic_a_msg)
</pre>
        </td>
    </tr>
</tbody></table>

Suppose a goal point at $(1.0,-1.5,0.0)$ has to be published to a topic `/goal` with the message type `geometry_msgs/msg/Point`, and the message is published by the node `Behavior`:

```python
# 1. IMPORT
import rclpy
from rclpy.node import Node
from geometry_msgs.msg import Point

# NODE CLASS
class Behavior(Node):
    

    def __init__(self):
        super().__init__('behavior')
        # 2. NODE PROPERTIES

        # 3. NODE HANDLES
        self.point_pub = self.create_publisher(Point, '/goal', 10)

    # 4. NODE CALLBACKS

    def some_function(self):
        # 5. HOW TO USE
        point_msg = Point()
        point_msg.x = 1.0
        point_msg.y = -1.5
        point_msg.z = 0.0
        self.point_pub.publish(point_msg)

# MAIN BOILER PLATE
def main(args=None):
    rclpy.init(args=args)
    node = Behavior() # this changed
    rclpy.spin(node)
    rclpy.shutdown()
if __name__ == '__main__':
    main()
```
Add the `geometry_msgs` library into `package.xml`.

## 2.4&emsp;Create a Basic Service Server
Suppose we want to receive service requests from the service `/service_a` with the service interface type `lib_a/srv/SrvTypeA`.
The following beginner Python syntax is suggested:
<table><tbody>
    <tr>
        <th></th>
        <th>Append To</th>
        <th>Code</th>
    </tr>
    <tr>
        <td>1.</td>
        <td>Import</td>
        <td>
        Import the message class:
        <pre lang="python">from lib_a.srv import SrvTypeA</pre>
        In <code>package.xml</code>, add the <code>&lt;depend&gt;</code> tag for the library.
<pre lang="xml">
&lt;depend&gt;lib_a&lt;/depend&gt;
</pre>
        </td>
    </tr>
    <tr>
        <td>2.</td>
        <td>Node Properties</td>
        <td>&mdash;</td>
    </tr>
    <tr>
        <td>3.</td>
        <td>Node Handles</td>
        <td>
            Initialize the handle to the service server...
            <ul>
                <li><code>SrvTypeA</code>: The imported interface class. </li>
                <li><code>'/service_a'</code>: The string of the service name. </li> 
                <li><code>self.service_a_callback</code>: The callback method is called when a request is received.</li>
            </ul>
<pre lang="python">
self.service_a_srv = self.create_service(SrvTypeA, '/service_a', self.service_a_callback)
</pre>
</td>
    </tr>
    <tr>
        <td>4.</td>
        <td>Node Callbacks</td>
        <td>The callback is run once a request from a client is received, and it returns a response.
<pre lang="python">
def service_a_callback(self, request, response):
    # ... fill up the response based on the request...
    return response
</pre>
        </td>
    </tr>
    <tr>
        <td>5.</td>
        <td>How to Use</td>
        <td>&mdash;</td>
    </tr>
</tbody></table>

Suppose a `Planner` node receives a request for a path between a start point and a goal point. The service interface type is `nav_msgs/srv/GetPlan`, and the service is `/get_plan`:
```python
# 1. IMPORT
import rclpy
from rclpy.node import Node
from nav_msgs.srv import GetPlan

# NODE CLASS
class Planner(Node):
    def __init__(self):
        super().__init__('behavior')
        # 2. NODE PROPERTIES

        # 3. NODE HANDLES
        self.get_plan_srv = self.create_service(GetPlan, '/get_plan', self.get_plan_callback)

    # 4. NODE CALLBACKS
    def get_plan_callback(self, request, response):
        path = some_path_planner(request.start, request.goal)
        response.plan = path
        return response

# MAIN BOILER PLATE
def main(args=None):
    rclpy.init(args=args)
    node = Planner() # this changed.
    rclpy.spin(node)
    rclpy.shutdown()
if __name__ == '__main__':
    main()
```

Add the `nav_msgs` library into the `package.xml`.

## 2.5&emsp;Create a Basic Service Client

A **service client** requests for a service.

In the official tutorial, when a service request is sent, all execution in the node is suspended (**blocking** / **asynchronous**) until the service response is received.
Blocking prevents the node from running other callbacks, such as an important timer loop that needs to be run at a high frequency.

To prevent blocking (**non-blocking** / **synchronous**) and allow simultaneous operations, threading (via executors or normal threads) can be used, or we can assign a **future** object.
The future object can be used to asynchronously check whether or not the response has been received, and what the response is when it is received.

Suppose we want to send a request over the `/service_a` service, which has the interface type `lib_a/srv/SrvTypeA`.
The following beginner Python syntax is suggested:
<table><tbody>
    <tr>
        <th></th>
        <th>Append To</th>
        <th>Code</th>
    </tr>
    <tr>
        <td>1.</td>
        <td>Import</td>
        <td>
        Import the message class:
        <pre lang="python">from lib_a.srv import SrvTypeA</pre>
        In <code>package.xml</code>, add the <code>&lt;depend&gt;</code> tag for the library.
<pre lang="xml">
&lt;depend&gt;lib_a&lt;/depend&gt;
</pre>
        </td>
    </tr>
    <tr>
        <td>2.</td>
        <td>Node Properties</td>
        <td><code>service_a_future</code>: A <b>future</b> is a common, concurrent computing concept. It is used to check if a response is received, and stores the response when received. See chapter on advanced services.
<pre lang="python">
service_a_future = None
</pre>
        </td>
    </tr>
    <tr>
        <td>3.</td>
        <td>Node Handles</td>
        <td>
            Initialize the handle to the client.
            <ul>
                <li><code>SrvTypeA</code>: The imported interface class. </li>
                <li><code>'/service_a'</code>: The string of the service name. </li> 
            </ul>
<pre lang="python">
self.service_a_cli = self.create_client(SrvTypeA, '/service_a')
</pre>
</td>
    </tr>
    <tr>
        <td>4.</td>
        <td>Node Callbacks</td>
        <td>&mdash;</td>
    </tr>
    <tr>
        <td>5.</td>
        <td>How to Use</td>
        <td>
            To <b>send a request</b>, fill up the service request and then call the service with the request. The future is assigned to monitor the response.
<pre lang="python">
if self.service_a_future is None:
    if request = SrvTypeA.Request()
    # ... fill service request...
    self.service_a_future = self.service_a_cli.call_async(request)
</pre>
            To <b>asynchronously wait for the response</b> so that other callbacks can be run:
<pre lang="python">
if self.service_a_future is not None:
    if self.service_a_future.done():
        # use the response at self.service_a_future.result()
</pre>
        </td>
    </tr>
</tbody></table>

Suppose a `Behavior` node wants to request for a path between a start point and a goal point. The service interface type is `nav_msgs/srv/GetPlan`, and the service is `/get_plan`.

A path finding request can be a few milliseconds long, which is time-consuming.
Therefore, the service request must be **non-blocking** especially if there is another frequently running callback (not shown below) that checks for obstacle safety (i.e. collision).

```python
# 1. IMPORT
import rclpy
from rclpy.node import Node
from nav_msgs.srv import GetPlan

# NODE CLASS
class Behavior(Node):
    def __init__(self):
        super().__init__('behavior')
        # 2. NODE PROPERTIES
        self.get_plan_future = None

        # 3. NODE HANDLES
        self.get_plan_cli = self.create_client(GetPlan, '/get_plan')

    # 4. NODE CALLBACKS

    # 5. HOW TO USE 
    def function_to_send_request(self, start, goal):    
        if self.get_plan_future is None:
            request = GetPlan.Request()
            request.start = start
            request.goal = goal
            request.tolerance = 0.0
            self.get_plan_future = self.get_plan_cli.call_async(request)

    def function_to_use_response(self):
        path = []
        if self.get_plan_future is not None:
            if self.get_plan_future.done():
                path = self.get_plan_future.result().plan
        return path

# MAIN BOILER PLATE
def main(args=None):
    rclpy.init(args=args)
    node = Behavior() # this changed.
    rclpy.spin(node)
    rclpy.shutdown()
if __name__ == '__main__':
    main()
```

Add the `nav_msgs` library into `package.xml`.

## 2.6&emsp;Read Parameters
In this section, we try to read a parameter from the same node. 
Nodes from other parameters can be accessed, but require service calls (see more in later chapters).

Like topics and services, parameters are strongly typed. The parameters can only be one of the following types:

| Type | Examples of Type | Get Parameter Property |
|-|-|-|
| **Boolean** | `True` or `False` | `.bool_value` |
| **Boolean array** | `[True`, `False`, `True]` | `.bool_array_value` | 
| **Byte array** | `bytes([0x1A, 0x00, 0x10])`| `.byte_array_value` |
| **Double** | `1.0` | `.double_value` |
| **Double array**  | `[1.0, 2.1, -3.0]` | `.double_array_value` |
| **Integer** | `1` | `.integer_value` |
| **Integer array** | `[1, 2, 3]` | `.integer_array_value` |
| **String** | `'hi'` | `.string_value` |
| **String array** | `['hi', 'i', 'am']` | `.string_array_value` | 

Dynamic typing of parameters is possible, but it is not a good practice as this can cause unexpected semantic errors. 

Suppose we want to access the parameter `param_a` in a node.
The following beginner Python syntax is suggested:
<table><tbody>
    <tr>
        <th></th>
        <th>Append To</th>
        <th>Code</th>
    </tr>
    <tr>
        <td>1.</td>
        <td>Import</td>
        <td>&mdash;</td>
    </tr>
    <tr>
        <td>2.</td>
        <td>Node Properties</td>
        <td>&mdash;</td>
    </tr>
    <tr>
        <td>3.</td>
        <td>Node Handles</td>
        <td>
            Declare the parameter, so the node knows that it needs to access this parameter in its life time.
            <ul>
                <li><code>'param_a'</code>: The name of the parameter </li>
                <li><code>default_value</code>: The initial value. The type of the parameter is inferred from this value. Therefore, if it is <code>1</code>, the type is inferred as an integer. If it is <code>1.0</code>, the type is inferred as a double. If <code>'1.0'</code>, the type is inferred as a string.</li> 
            </ul>
<pre lang="python">
self.param_a_param = self.declare_parameter('param_a', default_value)
</pre>
</td>
    </tr>
    <tr>
        <td>4.</td>
        <td>Node Callbacks</td>
        <td>&mdash;</td>
    </tr>
    <tr>
        <td>5.</td>
        <td>How to Use</td>
        <td>
            Depends on the type. See the preceding table for the methods. For example, to read an integer-typed parameter:
<pre lang="python">
# for integer parameter, use: 
# self.param_a_param.get_parameter_value().integer_value
# for others, see preceding table.
</pre>
        </td>
    </tr>
</tbody></table>

Suppose we want to read a double parameter called `speed`, which should have a default value of `2.0` meters per second if not set:
```python
# 1. IMPORT
import rclpy
from rclpy.node import Node

# NODE CLASS
class SomeNode(Node):
    

    def __init__(self):
        super().__init__('some_node')
        # 2. NODE PROPERTIES

        # 3. NODE HANDLES
        self.speed_param = self.declare_parameter('speed', 2.0)

    # 4. NODE CALLBACKS

    # 5. HOW TO USE 
    def some_function(self):  
        speed = self.speed_param.get_parameter_value().double_value

# MAIN BOILER PLATE
def main(args=None):
    rclpy.init(args=args)
    node = SomeNode() # this changed.
    rclpy.spin(node)
    rclpy.shutdown()
if __name__ == '__main__':
    main()
```

# 3&emsp;Tasks

1. Copy the Pythonic structure into the `tutorial.py` Python scripts for the `tutorial` node.

2. In the node constructor make sure the super class constructor contains the correct node name:
    ```python
    super().__init__('tutorial')
    ```

3. Create a **timer callback**.
    - The timer callback runs every 0.5s.
    - In the callback, increment a counter that starts from 0 at every call, and print the counter to the terminal. Make use of the `# 2. NODE PROPERTIES` location to implement the persistent counter.
    - When run, the output on the terminal should show the counter incrementing every 0.5s.

4. Do not remove the previous code. Now, create a **topic subscriber**.
    - The subscriber listens to the topic `/turtle1/pose`. 
    - Using the ROS2 CLI and a running `turtlesim` node, determine the message type in the topic.
    - As indicated in the template, the subscriber callback should copy the received message into the node.
    - In the **timer callback**, print out the $x$ and $y$ coordinates of the copied message using the following Python f-string:
        ```python
        # x = the x coordinate from the copied message
        # y = the y coordinate from the copied message
        print(f'({x}, {y})')
        ```
    - If the copied message is `None` (i.e. message has not been received), do not print the coordinates. An error will be thrown if you do so.
    - When run, the coordinates of the turtle should be printed to the terminal every 0.5s.

5. Do not remove the previous codes. Now, create a **topic publisher**.
    - The publisher publishes into the topic `/turtle1/cmd_vel`.
    - Using the ROS2 CLI and a running `turtlesim` node, determine the message type in the topic, and its data fields.
    - In the **timer callback**, publish a message that alternates  the linear $x$ velocity between `1.0` and `-1.0` at every call.
    - When run, the turtle should move back and forth every 0.5s on the `turtlesim` node.

6. Do not remove the previous codes. Now, create a **service client**.
    - The client will request to spawn a new turtle over the service `/spawn`.
    - Using the ROS2 CLI and a running `turtlesim` node, determine the service interface type for the service, and its data fields.
    - From the **timer callback**, send the service request **only once** throughout the entire run.
    - The request should spawn a new turtle at the coordinates $(1, 1)$ at any orientation. The name should be left blank using an empty string `''`.
    - Using the future object, print out the automatically assigned name of the new turtle in the **timer callback** after the response is received.

7. **[Q1]** Cut and paste the entire `tutorial.py` code on to the Canvas Quiz.
    
