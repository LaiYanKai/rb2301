10&emsp;Launch Files and Bash Scripts
==============

***RB2301 Robot Programming***

**&copy; Lai Yan Kai, National University of Singapore**

This chapter focuses on bulk running and launching nodes. The launch file and its command line interface, as well as some convenient bash scripts, are shown.

# Table of Contents

[1&emsp;Launch Files](#1launch-files)

&emsp;[1.1&emsp;Where to Place](#11where-to-place)

&emsp;[1.2&emsp;Installing with `setup.py`](#12installing-with-setuppy)

&emsp;[1.3&emsp;Command Line Interface](#13command-line-interface)

[2&emsp;Launch File Template](#2launch-file-template)

&emsp;[2.1&emsp;Package Installation Directories](#21package-installation-directories)

&emsp;[2.2&emsp;Launch Arguments](#22launch-arguments)

&emsp;[2.3&emsp;Launch other Launch Files](#23launch-other-launch-files)

&emsp;[2.4&emsp;Run Nodes](#24run-nodes)

&emsp;&emsp;[2.4.1&emsp;Namespace](#241namespace)

&emsp;&emsp;[2.4.2&emsp;Parameters](#242parameters)

&emsp;&emsp;[2.4.3&emsp;Remapping Topics](#243remapping-topics)

&emsp;&emsp;[2.4.4&emsp;Node Arguments](#244node-arguments)

&emsp;[2.5&emsp;Tasks](#25tasks)

[3&emsp;Bash Script](#3bash-script)

&emsp;[3.1&emsp;Where to Place](#31where-to-place)

&emsp;[3.2&emsp;How to Run](#32how-to-run)

&emsp;[3.3&emsp;Examples](#33examples)

&emsp;&emsp;[3.3.1&emsp;Build Script](#331build-script)

&emsp;&emsp;[3.3.2&emsp;Clean Build Script](#332clean-build-script)

&emsp;&emsp;[3.3.3&emsp;Run Script](#333run-script)

&emsp;&emsp;[3.3.4&emsp;Run Script with Arguments](#334run-script-with-arguments)

&emsp;&emsp;[3.3.5&emsp;Launch Script](#335launch-script)

# 1&emsp;Launch Files
Launch files can be used to run multiple nodes or start other launch files. All the nodes specified in the launch files are run simultaneously from a single terminal, and are handy for most projects, especially large ones.

Launch files are typically Python scripts ending with `.launch.py` in their file names, although they can be in `.xml` or `.yaml` formats. 
Python scripts are the norm because of their flexibility, and this section focuses only on Python launch files.

## 1.1&emsp;Where to Place

The launch files are usually located in the package where nodes to be run are located, but they can also appear in another package if the package depends on the current package.
Regardless of the case, the launch files can only be stored in the `launch` directory of a package. For example,

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
                            <summary><code>launch</code>&emsp; The folder containing launch files.</summary>
                            <dl>
                                <dd><code>launch_a.launch.py</code>&emsp; A launch file.</dd>
                            </dl>
                        </details></dd>
                        <dd><details>
                            <summary><code>pkg_a/</code></summary>
                            <dl>
                                <dd><code>__init__.py</code></dd>
                                <dd><code>node_a.py</code>&emsp;A node.</dd>
                                <dd><code>node_b.py</code>&emsp;Another node.</dd>
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


## 1.2&emsp;Installing with `setup.py`

The launch file needs to be available in the installation directory after building, so that ROS2 can find it. 
The following code **should only be inserted once** to install the `launch` directory and its contents.

In `setup.py`, 
1. Import `glob`.
2. Look for the `data_files=` keyword argument, and add the following line:
    ```python
    from setuptools import find_packages, setup
    from glob import glob

    package_name = 'pkg_a' 

    setup(
        #...
        data_files=[
            #...
            ('share/' + package_name + '/launch', glob('launch/*.launch.py')),
        ]
        #...
    )
    ```
3. Build with `colcon build --symlink-install`. Unless the name of the launch file changes, or a new launch file is created, re-building is not necessary.

## 1.3&emsp;Command Line Interface

Use `ros2 launch` to launch the launch files.

| | Command | Description |
| - | - | - |
| 1. | `ros2 launch -h` | Use the help messages. |
| 2. | `ros2 launch pkg_a launch_a.launch.py` | Launches a launch file `launch_a.launch.py` from the package `pkg_a`. |
| 3. | `ros2 launch pkg_a launch_a.launch.py arg_a:='a value'` | Launches the launch file with the launch argument `arg_a` set at the value `'a value'` |
| 4. | `ros2 launch pkg_a launch_a.launch.py arg_a:='a value' arg_b:=2` | Launches the launch file with multiple launch arguments specified. |


# 2&emsp;Launch File Template
The Pythonic template for a launch file that can accept arguments, run nodes, and launch other launch files, is shown below.

Note that the launch file **does not immediately parse the code**. It waits until the `generate_launch_description()` function returns before parsing. 
Therefore, it is not possible to **print any of the substitutions** for troubleshooting. 
This is primarily to preserve the legacy behavior of launch files in ROS1 when `.xml` launch files are used.

```python
from launch import LaunchDescription
from launch.actions import IncludeLaunchDescription, DeclareLaunchArgument
from launch.launch_description_sources import PythonLaunchDescriptionSource
from launch.substitutions import LaunchConfiguration, PathJoinSubstitution
from launch_ros.actions import Node
from launch_ros.substitutions import FindPackageShare

def generate_launch_description():
    # initialize launch description
    ld = LaunchDescription()

    # 1. PACKAGE INSTALLATION DIRECTORIES 

    # 2. LAUNCH ARGUMENTS
    
    # 3. LAUNCH OTHER LAUNCH FILES
    
    # 4. RUN NODES

    return ld

```
Maybe the ROS2 developers will streamline the different functions of the launch file in the future.

## 2.1&emsp;Package Installation Directories
A package installation directory contains the path to the installed `share` directory of a package after the package is built.

The installation directory of a package `pkg_a` can be substituted as:
```python
pkg_pkg_a = FindPackageShare('pkg_a')
```
As a substitution, it is not possible to print the value, as the parsing occurs only after `generate_launch_description()` function returns.

## 2.2&emsp;Launch Arguments
A launch file can contain launch arguments, which can be used to modify the behavior of the launch file, just like ROS parameters.

1. Declare the launch argument `arg_a` with the `DeclareLaunchArgument()` subtitution. The initial value of the argument must be specified in `default_value=`. **The argument value can only be specified as a string**. `description=` may be used to describe the argument for troubleshooting purposes.

    ```python
    arg_arg_a = DeclareLaunchArgument(
        'arg_a', 
        default_value='True',
        description='some kind of description'
    )
    ld.add_action(arg_arg_a)
    ```

2. Then, use the argument somewhere else in the file by using `LaunchConfiguration()` substitution.

    ```python
    LaunchConfiguration('arg_a')
    ```
Note that all of the substitutions cannot be printed.

## 2.3&emsp;Launch other Launch Files

A launch file can launch other launch files, either from the same package or another package. 

The example below shows how to launch a launch file `launch_file_a.launch.py` from another package `pkg_b` while substituting one of the launch file's arguments `arg_c` with the current file's argument `arg_b`.
```python
# 1. PACKAGE INSTALLATION DIRECTORIES
pkg_pkg_b = FindPackageShare('pkg_b')

# 2. LAUNCH ARGUMENTS
arg_arg_b = DeclareLaunchArgument(
    'arg_b', 
    default_value='[1,2,3]',
    description='some kind of description'
)
ld.add_action(arg_arg_b)

# 3. LAUNCH OTHER LAUNCH FILES
launch_launch_file_a = IncludeLaunchDescription(
    PythonLaunchDescriptionSource([
        PathJoinSubstitution([
            pkg_pkg_b, 'launch', 'launch_file_a.launch.py'
        ])
    ]),
    launch_arguments={
        'arg_a': 'value_a',
        'arg_c': LaunchConfiguration('arg_b'),
    }.items()
)
ld.add_action(launch_launch_file_a)
```

## 2.4&emsp;Run Nodes
The example below shows how to run a node in the executable `node_a` (same as when `ros2 run` is used) from a package `pkg_a` from the launch file.
```python
# 4. RUN NODES
node_node_a = Node(
    package='pkg_a',
    executable='node_a',
    output='screen',
    emulate_tty=True,
)
ld.add_action(node_node_a)
```
### 2.4.1&emsp;Namespace
To launch the node in a namespace `namespace_a`,
```python
# 4. RUN NODES
node_node_a = Node(
    package='pkg_a',
    executable='node_a',
    output='screen',
    emulate_tty=True,
    namespace='namespace_a',
)
ld.add_action(node_node_a)
```
### 2.4.2&emsp;Parameters
To include a parameter file `params_a.yaml` from the same package:
```python
# 1. PACKAGE INSTALLATION DIRECTORIES
pkg_pkg_a = FindPackageShare('pkg_a')

# 4. RUN NODES
node_node_a = Node(
    package='pkg_a',
    executable='node_a',
    output='screen',
    emulate_tty=True,
    parameters=[
        PathJoinSubstitution([pkg_pkg_a, 'params', 'params_a.yaml'])
    ],
)
ld.add_action(node_node_a)
```
To include parameters without using a parameter file,
```python
# 4. RUN NODES
node_node_a = Node(
    package='pkg_a',
    executable='node_a',
    output='screen',
    emulate_tty=True,
    parameters=[
        {
            'string_a': 'a string',
            'color.blue': 0
        }
    ],
)
ld.add_action(node_node_a)
```

### 2.4.3&emsp;Remapping Topics
To remap a topic `/topic_a` used by `node_a` to `/topic_b`,
```python
# 4. RUN NODES
node_node_a = Node(
    package='pkg_a',
    executable='node_a',
    output='screen',
    emulate_tty=True,
    remappings=[
        ('/topic_a', '/topic_b'),
    ],
)
ld.add_action(node_node_a)
```

### 2.4.4&emsp;Node Arguments
Arguments are not ROS parameters. 
These are the same command line arguments that trail the `ros2 run pkg_a node_a` syntax. 
Each *space* in the original command should be delimited. 

For example, if we want to mimic the command `ros2 run pkg_a node_a --ros-args -p string_a:='a string'`,
```python
# 4. RUN NODES
node_node_a = Node(
    package='pkg_a',
    executable='node_a',
    output='screen',
    emulate_tty=True,
    arguments=[
        '--ros-args', 
        '-p', 
        'string_a:="a string"'
    ]
)
ld.add_action(node_node_a)
```

## 2.5&emsp;Tasks

1. Download the package-level `rb2301_p1.zip` file containing the `rb2301_description` and `rb2301_bringup` C++ packages from Canvas. Extract them into the `src` directory.

2. Create a launch file `sim.launch.py` in the `rb2301_tutorial` package.

    **[Q2a]** What is the code to add to `setup.py` so that the launch folder and its contents are installed? Remember to build.

3. In the launch file, create a new launch argument `world` with the default value `test.sdf`.

    **[Q2b]** What is the code to create the argument?

4. In the launch file, insert code to launch the similarly-named `sim.launch.py` launch file from the `rb2301_bringup` package. 
The `rb2301_bringup`'s launch file has a similarly-named `world` argument, which should take the value of the current launch file's `world` argument.

    **[Q2c]** What is the code to launch this launch file with the `world` argument substituted with the current file's `world` argument?

5. Launch the `sim.launch.py` launch file from the `rb2301_tutorial` package. This should launch Gazebo with a robot in it, along with some basic shapes as obstacles.

    **[Q2d]** What is the CLI to launch this file?

6. Launch the `sim.launch.py` launch file from the `rb2301_tutorial` package, with the `world` argument set to `empty.sdf` using the CLI. This should launch Gazebo with the robot and without obstacles.

    **[Q2e]** What is the CLI to launch this file with the argument?

7. Create a node called `sim` that can be run with `ros2 run rb2301_tutorial sim`. 

    1. Subscribe to the `/odom` topic, which has a `nav_msgs/msg/Odometry` message type.

    2. Create a timer callback that loops every 0.2 seconds and that prints the robot's $x$ and $y$ coordinates from the `/odom` topic.

    **[Q2f]** Cut and paste the contents of the Python file.

8. In `sim.launch.py` from the `rb2301_tutorial` package, run the node `sim`.

    **[Q2g]** What is the code to run this node?

# 3&emsp;Bash Script
A bash script is a script that uses the Ubuntu terminal syntax to automate commands.
This section shows how to shorten commonly used commands with bash scripts.

## 3.1&emsp;Where to Place

The bash scripts shown in this section should be stored under the workspace directory since they are not specific to a package.
<table><tbody><tr><td>
    <details open>
    <summary><code>workspace_a/</code>&emsp;Workspace.</summary>
    <dl>
        <dd><details> 
            <summary><code>src/</code></summary>
            <dl><dd>...</dd></dl>
        </details></dd>
        <dd><code>bash_script.sh</code>&emsp; A bash script</dd>
    </dl>
    </details>
</td></tr></tbody></table>

## 3.2&emsp;How to Run

Make sure to assign permissions to the file using `chmod`. Permissions only need to be **assigned once** unless the file is renamed or replaced:
```bash
cd workspace_a
chmod +x bash_script.sh
```

Run the bash script from the workspace with the following command, replacing `bash_script` with the relevant script name:

```bash
cd workspace_a          # cd to the workspace containing the script
./bash_script.sh   # run the bash script
```

The `./` command opens a new, background "terminal" to run the code. Therefore, any environment variables set in the current terminal, like those set by `install/setup.bash`, are not copied to the new terminal. 
This is particularly useful for avoiding execution errors.

The output of the background "terminal" is still piped into the current terminal.

## 3.3&emsp;Examples
The following shows examples of bash scripts that wrap commonly used commands. It is up to you to rename the files, or insert commands. 

### 3.3.1&emsp;Build Script

To shorten the build command, in a bash script `bd.sh` (name is up to you), type the following:
```bash
#!/bin/bash
cd workspace_a
colcon build --symlink-install
```

### 3.3.2&emsp;Clean Build Script

To cleanly build the workspace, in a bash script `clean_bd.sh` (name is up to you):
```bash
#!/bin/bash
cd workspace_a
colcon build --symlink-install --cmake-clean-first
```

### 3.3.3&emsp;Run Script

To run a node, in a bash script `run.sh` (name is up to you):
```bash
#!/bin/bash
source workspace_a/install/setup.bash
ros2 run pkg_a node_a # and some other ros2 arguments
```

### 3.3.4&emsp;Run Script with Arguments
If a different node needs to be run at different times, arguments can be used. 
The first argument is substituted into `$1`, and the second to `$2` etc. 
`"#@"` can be used to pass all the arguments without losing any original quote marks.

1. To use **positional arguments**, use `$1`, `$2`, etc.
    Suppose `run.sh` contains:
    ```bash
    #!/bin/bash
    source workspace_a/install/setup.bash
    ros2 run $1 $2
    ```
    The script can be run with:
    ```bash
    cd workspace_a
    ./bd.sh pkg_a node_b 
    ```
2. To pass **all arguments**, use `"$@"`.
    Suppose `run.sh` contains:
    ```bash
    source workspace_a/install/setup.bash
    ros2 run pkg_a "$@"
    ```
    which can be run with the following to pass in some parameters:
    ```bash
    ./run.sh node_b --ros-args -p string_a:='this string' -p color.blue:=2
    ```

### 3.3.5&emsp;Launch Script

To launch nodes from a launch file, in a bash script `launch.sh` (name is up to you):
```bash
source workspace_a/install/setup.bash
ros2 launch pkg_a launch_file_a.launch.py
```
