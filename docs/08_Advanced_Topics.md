8&emsp;ROS2 Topics with Quality of Service
==============

***RB2301 Robot Programming***

**&copy; Lai Yan Kai, National University of Singapore**

To write a basic topic subscriber or publisher using the Python client library, please refer to [05_Node.md](05_Node.md#22create-a-basic-topic-subscriber).
This section focuses primarily on using Quality of Services (QoS, see [07_Quality_of_Service.md](07_Quality_of_Service.md)) to create advanced topics.

# Table of Contents
[1&emsp;Tasks](#1tasks)

[2&emsp;Sensor Data and Preset QoS Profile](#2sensor-data-and-preset-qos-profile)

[3&emsp;Latched Topic and Custom Profile](#3latched-topic-and-custom-profile)

[4&emsp;Shallow Depth Policy for Busy Subscribers](#4shallow-depth-policy-for-busy-subscribers)

[5&emsp;Ensure Messages Arrive Before Use](#5ensure-messages-arrive-before-use)

[6&emsp;Compatibility of Policies](#6compatibility-of-policies)

&emsp;[6.1&emsp;Durability Policy](#61durability-policy)

&emsp;[6.2&emsp;Reliability Policy](#62reliability-policy)

# 1&emsp;Tasks

1. Create a `publishers.py` file containing a `Publishers` node class. This Python script should be executable with `ros2 run rb2301_tutorial pubs`.
    
    ```python
    # rclpy
    from rclpy.node import Node
    from std_msgs.msg import Header

    class PublishersNode(Node):

        sensor_pub = None
        latch_pub = None

        def __init__(self):
            super().__init__('publishers')

            self.timer = self.create_timer(
                0.05, self.timer_callback)
            self.i = 0

        def timer_callback(self):
            self.i += 1
            msg = Header()
            msg.stamp = self.get_clock().now().to_msg()
            msg.frame_id = str(self.i)

            print(f'Publish: {self.i}')
            
            if self.sensor_pub is not None:
                self.sensor_pub.publish(msg)

            if self.latch_pub is not None:
                self.latch_pub.publish(msg)
        
    def main(args=None):
        rclpy.init(args=args)
        node = PublishersNode()
        rclpy.spin(node) 
        rclpy.shutdown()
    if __name__ == '__main__':
        main()
    ```

2. Create a `subscribers.py` file containing a `Subscribers` node class. This Python script should be executable with `ros2 run rb2301_tutorial subs`.

    ```python
    # rclpy
    from rclpy.node import Node
    from rclpy.time import Time
    from rclpy.duration import Duration
    from std_msgs.msg import Header

    class SubscribersNode(Node):

        latch_msg = None
        sensor_msg = None
        shallow_msg = None

        def __init__(self):
            super().__init__('subscribers')

            # how long the timer_callback will 
            # block other callbacks for (sec)
            self.timer_wait = 0.0

            self.timer = self.create_timer(0.1, self.timer_callback)

        def latch_sub_callback(self, msg):
            self.latch_msg = msg
            print(f'Received       Latch: id({msg.frame_id})')
            
        def sensor_sub_callback(self, msg):
            self.sensor_msg = msg
            print(f'Received      Sensor: id({msg.frame_id})')

        def shallow_sub_callback(self, msg):
            self.shallow_msg = msg
            print(f'Received     Shallow: id({msg.frame_id})')

        def get_elapsed(self, msg):
            now = self.get_clock().now()
            then = Time.from_msg(msg.stamp)
            return float((now - then).nanoseconds) / 1e9
        
        def print_msg(self, name, msg):
            if msg is not None:
                id = msg.frame_id
                elapsed = self.get_elapsed(msg)
                print(f'{name}: id({id}) lifespan({elapsed} sec)')
            
        def timer_callback(self):
            print('---- Timer sees: ---')
            self.print_msg('      Latch', self.latch_msg)
            self.print_msg('     Sensor', self.sensor_msg)
            self.print_msg('    Shallow', self.shallow_msg)
            print('---- ')

            # mimic some complex calculations which  
            # prevents this callback from returning, 
            # blocking other callbacks.
            self.get_clock().sleep_for(
                Duration(seconds=self.timer_wait))

    def main(args=None):
        rclpy.init(args=args)
        node = SubscribersNode()
        rclpy.spin(node) 
        rclpy.shutdown()
    if __name__ == '__main__':
        main()

    ```

3. Remember to build:
    
    ```bash
    cd ~/rb2301
    colcon build --symlink-install
    ```

4. Answer the questions in their respective sections.

# 2&emsp;Sensor Data and Preset QoS Profile

In this section, we make use of a preset QoS profile to create a topic suitable for transmitting sensor data.
With this profile, messages are allowed to be lost in a lossy network between a robot and a remote computer. 
In other words, no attempts are made to re-send the lost messages, ensuring that the most recent sensor data are received more frequently.

1. **[Q1a]** What is the **preset profile** for transmitting sensor data, which looks like `qos_profile_a` and can be imported from `rclpy.qos`?
2. **[Q1b]** What is the name of the QoS **policy** that when adjusted, allows messages to be lost in a lossy network?
3. **[Q1c]** What is the **value** of this policy, that allows messages to be lost in a lossy network? Verify that the preset profile uses this value.

4. In `publishers.py`:
    1. Import the profile. Replace `qos_profile_a`:
        
        ```python
        from rclpy.qos import (
            qos_profile_a
        )
        ```
    
    2. In the node constructor, create the `sensor_pub` publisher handle that uses the profile. Replace `qos_profile_a`:
        
        ```python
        self.sensor_pub = self.create_publisher(
            Header, '/sensor', qos_profile_a)
        ```

5. In `subscribers.py`: 
    1. Import the profile.

    2. In the node constructor, create the `sensor_sub` subscriber handle that uses the profile. Replace `qos_profile_a`:

        ```python
        self.sensor_sub = self.create_subscription(
                Header, '/sensor', self.sensor_sub_callback, qos_profile_a)
        ```

# 3&emsp;Latched Topic and Custom Profile

In this section, we use a custom QoS profile to implement a latched topic.
A latched connection persists some published messages so that late-joining subscribers will be able to receive the past messages.

1. **[Q2a]** What is the name of the QoS **policy** that allows published messages to persist for late joining subscribers? 

2. **[Q2b]** What is the **value** that this QoS policy should have to persist the messages?

3. In `publishers.py`:
    1. Import the correct policy by replacing `PolicyA` in the code below. You may add on to the previous import statement by using commas:

        ```python
        from rclpy.qos import (
            QoSProfile,
            HistoryPolicy,
            PolicyA
        )
        ```
    2. In the node constructor, create the custom QoS policy `qos_profile_latch`. Replace `PolicyA`, `VALUE_A`, and the keyword argument `policy_a`:

        ```python 
        qos_profile_latch = QoSProfile(
            history=HistoryPolicy.KEEP_LAST,
            depth=5,
            policy_a=PolicyA.VALUE_A
        )
        ```

    3. In the node constructor, create the publisher handle `latch_pub` that uses the custom QoS profile:

        ```python
        self.latch_pub = self.create_publisher(
            Header, '/latch', qos_profile_latch
        )
        ```

4. In `subscribers.py`:
    1. Import the policy like the publisher.
    2. In the node constructor, copy and paste the custom `qos_profile_latch` from the publisher.
    3. In the node constructor, create the subscriber handle `latch_sub` that uses the profile:

        ```python
        self.latch_sub = self.create_subscription(
            Header, '/latch', self.latch_sub_callback, qos_profile_latch
        )
        ```

4. In terminal `A`, first run the publishers node:

    ```bash
    cd ~/rb2301
    source install/setup.bash
    clear
    ros2 run rb2301_tutorial pubs
    ```

5. Then, in terminal `B`, run the subscribers node:

    ```bash
    cd ~/rb2301
    source install/setup.bash
    clear
    ros2 run rb2301_tutorial subs
    ```

7. After about a second, `Ctrl+C` in both terminals `A` and `B`.

8. **[Q2c]** In terminal `B`, scroll to the top and determine the number of past messages that were received by the subscriber when the subscriber is first run. 

9. **[Q2d]** Verify that this number agrees with two other policies in the profile. These policies will determine the number of past messages persisted by the publisher (provided that both subscriber and publisher policies are identical). What are these two policies?

# 4&emsp;Shallow Depth Policy for Busy Subscribers

In this section, we modify the depth policy for an existing QoS profile to account for busy subscribers.
A node that subscribes to a topic may be too busy to receive and process all messages. 
This can occur when a poorly designed, slow-running callback begins to block subscriber callbacks, causing only the oldest message in the subscriber queue to be processed.
By keeping the depth shallow, the queue can be shortened, so that the last message in queue tends to be more recent. 

1. In `subscribers.py`, in the node constructor, create a custom profile `qos_profile_shallow` that has the same definition as the sensor data preset.

    ```python
    qos_profile_shallow = QoSProfile(
        # history=?
        # depth=?
        # durability=?
        # reliability=?
    )
    ```

2. In the same node constructor, create a subscriber handler that subscribes to the `/sensor` topic and uses the `qos_profile_shallow` profile:

    ```python
    self.shallow_sub = self.create_subscription(
        Header, '/sensor', self.shallow_sub_callback, qos_profile_shallow
    )
    ```

3. In the same node constructor, increase the waiting time in the timer callback to mimic long computation time. The "computation time" of 1 second far exceeds the expected rate of 0.1 seconds for the timer callback.

    ```python
    self.timer_wait = 1.0
    ```

4. In terminals `A` and `B`, run both nodes and verify that the same message is copied by the shallow and sensor handles before every timer callback. Notice the lifespan, which is the time since publication. `Ctrl+C` on both terminals when done.

5. Now, adjust the depth in `qos_profile_shallow`, so that the message copied by the shallow handle is likely to be more recent. Run the nodes to verify. 

6. **[Q3]** Determine the smallest depth so that the most recent message is likely to be copied. The answer may differ from the ideal value due to unexpected behavior from the middleware.

# 5&emsp;Ensure Messages Arrive Before Use
It is good practice to check if a message has arrived before a callback uses it. Otherwise, unintended behaviors may occur or errors may be thrown. 

The simplest way to do so is to initialize the copied messages as `None`, and using the `if` statement to check the copied messages:

```python
from rclpy.node import Node
from std_msgs.msg import String

class SomeNodeA(Node):

    topic_a_msg = None # message starts off as `None`

    def __init__(self):
        super.__init__('some_node_a')

        self.topic_a_sub = self.create_subscription(
            String, '/topic_a', self.topic_a_sub_callback)

        self.timer = self.create_timer(0.5, self.timer_callback)
        
    def topic_a_sub_callback(self, msg):
        self.topic_a_msg = msg # message is no longer `None` once a message arrives.

    def timer_callback(self):
        if self.topic_a_msg is not None:
            print(self.topic_a_msg) # do something when message arrives
        # if topic_a_msg is None:
        #    pass # do something else if message has not arrived

def main(args=None):
    rclpy.init(args=args)
    node = SomeNodeA()
    rclpy.spin(node)
    rclpy.shutdown()
if __name__ == '__main__':
    main()
```

# 6&emsp;Compatibility of Policies

As mentioned in [07_Quality_of_Service.md](07_Quality_of_Service.md), the publisher and subscriber must have compatible policies in order for messages to be sent.
The compatibility for the durability and reliability policies are described in this section.

## 6.1&emsp;Durability Policy

The publisher and subscriber must have compatible durability policies in order for messages to be sent:

| Publisher | Subscriber | Compatibility |
|-|-|-|
| `TRANSIENT_LOCAL` | `TRANSIENT_LOCAL` | Yes |
| `VOLATILE` | `VOLATILE` | Yes |
| `TRANSIENT_LOCAL` | `VOLATILE` | Yes | 
| `VOLATILE` | `TRANSIENT_LOCAL` | No | 

If the subscriber has a more demanding policy than the publisher, communication becomes impossible. If the subscriber demands published messages to be latched (`TRANSIENT_LOCAL`) but the publisher does not do so (`VOLATILE`), communication does not occur.


1. Change the `durability=` keyword when initializing `qos_profile_latch`, so that the publisher's durability policy is set to `VOLATILE` and the subscriber's durability policy is set to `TRANSIENT_LOCAL`.

2. Run the publishers node in terminal `A` first.

3. Run the subscribers node in terminal `B` next.

4. `Ctrl+C` both terminals after about a second.

5. **[Q4a]** What is the full warning message from terminal `A`? 

6. **[Q4b]** What is the full warning message from terminal `B`?

7. **[Q4c]** Rerun the nodes for the other three pairs of policies. Among the four pairs of policies, determine the pair(s) of policies where the subscriber is **unable** to receive **latched** messages?

## 6.2&emsp;Reliability Policy
The publisher and subscriber must have compatible reliability policies in order for messages to be sent:

| Publisher | Subscriber | Compatibility |
|-|-|-|
| `BEST_EFFORT` | `BEST_EFFORT` | Yes |
| `RELIABLE` | `RELIABLE` | Yes |
| `RELIABLE` | `BEST_EFFORT` | Yes | 
| `BEST_EFFORT` | `RELIABLE` | No | 

If the subscriber expects the publisher to resend lost messages (`RELIABLE`) but the publisher refuses to do so (`BEST_EFFORT`), communication is not allowed.

1. Change the `reliability=` keyword while initializing `qos_profile_latch` so that communication becomes impossible due to incompatible reliability policies.

2. Run the publishers node in terminal `A`.

3. Run the subscribers node in terminal `B`.

4. `Ctrl+C` both terminals after about a second.

5. **[Q5a]** What is the full warning message from terminal `A`? 

6. **[Q5b]** What is the full warning message from terminal `B`?