04&emsp;Setting up a ROS2 Project
==============

***RB2301 Robot Programming***

**&copy; Lai Yan Kai, National University of Singapore**

In the previous chapter, we investigated the CLI to understand the ROS2 concepts and what it can do to introspect a ROS2 system. 
Before we can begin to use the Python client libraries to code, we need to know how to set up the project.

A project is made up of a **workspace** directory (folder), where all the project files are kept.
The workspace contains a `src` directory where the source code is kept.
A **package**, which contains source code that implements a major component or sub-system in a robotic system, is located in the `src` directory.
The source code for a ROS2 **node**, which implements a component of a robotic system, is implemented within the packages.

We explore two ways to create a project from scratch. The first is via the CLI, and the second a procedure that lists the minimal steps to do so.
The CLI cannot be used to initialize large projects or multiple nodes, so the procedure is important.

# Table of Contents

[1&emsp;Python vs C++](#1python-vs-c)

[2&emsp;Create Project From CLI](#2create-project-from-cli)

&emsp;[2.1&emsp;Create Workspace and `src` Directories](#21create-workspace-and-src-directories)

&emsp;[2.2&emsp;Create Python Package And Node](#22create-python-package-and-node)

[3&emsp;The Minimal Procedure](#3the-minimal-procedure)

&emsp;[3.1&emsp;Files and Directory](#31files-and-directory)

&emsp;[3.2&emsp;`package.xml`](#32packagexml)

&emsp;[3.3&emsp;`setup.cfg`](#33setupcfg)

&emsp;[3.4&emsp;`setup.py`](#34setuppy)

&emsp;[3.5&emsp;Multiple Nodes in a Package](#35multiple-nodes-in-a-package)

&emsp;[3.6&emsp;Multiple Packages](#36multiple-packages)

[4&emsp;Building and Running a Project](#4building-and-running-a-project)

&emsp;[4.1&emsp;Normal Build](#41normal-build)

&emsp;[4.2&emsp;Clean Build](#42clean-build)

&emsp;[4.3&emsp;Re-building](#43re-building)

&emsp;[4.4&emsp;Running After Building](#44running-after-building)

[5&emsp;Moving Projects](#5moving-projects)

&emsp;[5.1&emsp;Keep Only `src`](#51keep-only-src)

&emsp;[5.2&emsp;`.gitignore` for GitHub](#52gitignore-for-github)

[6&emsp;Tasks](#6tasks)

# 1&emsp;Python vs C++

ROS2 packages can be developed in Python or C++. 
The following table briefly describes the strengths and use cases of Python and C++.
<table><tbody>
    <tr>
        <th></th>
        <th>Compare</th>
        <th>Python</th>
        <th>C++</th>
    </tr>
    <tr>
        <td>1.</td>
        <td><b>Learning Curve (Debatable)</b></td>
        <td>Easy to most.</td>
        <td>Difficult to most.</td>
    </tr>
    <tr>
        <td>2.</td>
        <td><b>Optimization</b></td>
        <td>Little to None.</td>
        <td>Extensive.</td>
    </tr>
    <tr>
        <td>3.</td>
        <td><b>Use Case</b></td>
        <td>Used for quickly putting together simple implementations or library functions that run highly optimized code.</td>
        <td>Used for implementing complex algorithms and tasks.</td>
    </tr>
</tbody></table>

When coding, keep in mind the following best practices:
<table><tbody>
    <tr>
        <th></th>
        <th>Best Practice</th>
        <th>Description</th>
    </tr>
    <tr>
        <td>1.</td>
        <td><b>Ensure readability</b></td>
        <td>Keep the code readable by using comments and using variable and function names that are descriptive. For other conventions, google <code>PEP 8</code>.</td>
    </tr>
    <tr>
        <td>2.</td>
        <td><b>Use libraries</b></td>
        <td>As Python has little optimizations, rely on library functions as much as possible to manipulate data.</td>
    </tr>
    <tr>
        <td>3.</td>
        <td><b>Avoid micro-optimizations</b></td>
        <td>Avoid writing low-level code or using too much code for a tiny benefit in speed.</td>
    </tr>
    <tr>
        <td>4.</td>
        <td><b>Profile</b></td>
        <td>Record execution times to reliably conclude if one way is truly faster than the other.</td>
    </tr>
</tbody></table>

# 2&emsp;Create Project From CLI

In this section, we show how a project can be created from scratch using the ROS2 CLI.

## 2.1&emsp;Create Workspace and `src` Directories

The `src` directory in the workspace contains the source code of the project. By substituting `workspace_a` with the appropriate path to the workspace directory, create the `src` directory and the `workspace_a` directory with:
```bash
mkdir -p workspace_a/src
```

The following directory structure should now be observed:

<table><tbody><tr><td>
    <details open>
        <summary><code>workspace_a/</code>&emsp;The workspace directory.</summary>
        <dl>
            <dd><code>src/</code>&emsp;The directory containing source code.</dd>
        </dl>
    </details>
</td></tr></tbody></table>

## 2.2&emsp;Create Python Package And Node
A package implements a sub-system of a large robotic project, while a node implements a component of this sub-system.
Let the new Python package be called `py_pkg_a`, which contains a node that will be implemented in a Python file with the file name `filename_a`. 
For now, let's treat the filename as the same as the node name.

Navigate into the `src` folder, and use the ROS2 CLI to create the package and node at one go: 
```bash
cd workspace_a/src

ros2 pkg create --build-type ament_python --node-name filename_a py_pkg_a
```
After creating the package, the directory structure will look like:
<table><tbody><tr><td>
    <details open>
        <summary><code>workspace_a/</code>&emsp;The workspace directory.</summary>
        <dl>
            <dd><details open> 
                <summary><code>src/</code>&emsp;Contains packages.</summary>
                <dl>
                    <dd><details open>
                        <summary><code>py_pkg_a/</code>&emsp;A Python package</summary>
                        <dl>
                            <dd><details open>
                                <summary><code>py_pkg_a/</code>&emsp;Contains Python scripts created by the developer. Same name as package.</summary>
                                <dl>
                                    <dd><code>__init__.py</code>&emsp;Empty file. Can be written with variables that will be visible to all scripts in this package.</dd>
                                    <dd><code>filename_a.py</code>&emsp;The Python script for the node to be implemented.</dd>
                                </dl>
                            </details></dd>
                            <dd><details open>
                                <summary><code>resource/</code>&emsp;Folder containing ROS2 marker file.</summary>
                                <dl>
                                    <dd><code>py_pkg_a</code>&emsp;Marker file used by ROS2 to find the package. Empty file with no extension.</dd>
                                </dl>
                            </details></dd>
                            <dd><details open>
                                <summary><code>test/</code>&emsp;Contains automated test scripts. Can be deleted.</summary>
                                <dl>
                                    <dd><code>test_copyright.py</code></dd>
                                    <dd><code>test_flake8.py</code></dd>
                                    <dd><code>test_pep257.py</code></dd>
                                </dl>
                            </details></dd>
                            <dd><code>package.xml</code>&emsp;Manifest file. Tells ROS2 about which other ROS2 packages this package depends on.</dd>
                            <dd><code>setup.cfg</code>&emsp;Declarative configuration file. Tells ROS2 where to find the installed or testing files.</dd>
                            <dd><code>setup.py</code>&emsp;Distribution script. Tells ROS2 about the executables created from this package and their dependencies.</dd>
                        </dl>
                    </details></dd>
                </dl>
            </details></dd>
        </dl>
    </details>
</td></tr></tbody></table>

# 3&emsp;The Minimal Procedure

In this section, the minimal steps to setup a ROS2 Python script is shown here. 
The procedure essentially duplicates an existing project and edits the appropriate files, allowing us to more flexibly create multiple nodes and packages.

## 3.1&emsp;Files and Directory

The minimal directory structure is shown below. There is one workspace directory `workspace_a`, one Python package `py_pkg_a`, and one ROS2 node to be implemented in the Python file `filename_a.py`.

While using the template, 
- Ensure that all the files and directories exist. There can be additional files and directories.
- Substitute the 1x `workspace_a`, 3x `py_pkg_a` and 1x `filename_a` with your chosen names.
- Edit the `package.xml`, `setup.cfg`, and `setup.py` files. See later sections.

<table><tbody><tr><td>
    <details open>
        <summary><code>workspace_a/</code>&emsp;Workspace.</summary>
        <dl>
            <dd><details open> 
                <summary><code>src/</code></summary>
                <dl>
                    <dd><details open>
                        <summary><code>py_pkg_a/</code>&emsp;Python package.</summary>
                        <dl>
                            <dd><details open>
                                <summary><code>py_pkg_a/</code></summary>
                                <dl>
                                    <dd><code>__init__.py</code>&emsp;Python script that can be empty.</dd>
                                    <dd><code>filename_a.py</code>&emsp;Python script for a node.</dd>
                                </dl>
                            </details></dd>
                            <dd><details open>
                                <summary><code>resource/</code></summary>
                                <dl>
                                    <dd><code>py_pkg_a</code>&emsp;Marker file that is empty and has no file extension.</dd>
                                </dl>
                            </details></dd>
                            <dd><code>package.xml</code>&emsp;Manifest file. To edit package name inside.</dd>
                            <dd><code>setup.cfg</code>&emsp;Declarative configuration file.</dd>
                            <dd><code>setup.py</code>&emsp;Distribution script. To edit package name inside.</dd>
                        </dl>
                    </details></dd>
                </dl>
            </details></dd>
        </dl>
    </details>
</td></tr></tbody></table>

## 3.2&emsp;`package.xml`
The manifest file contains information about a package's ROS2 dependencies and distribution information. 

When using the template,
- Ensure that the tags `<...>` `</...>` below exist. There can be additional tags.
- Substitute the 1x `py_pkg_a`. 
- **[Optional]** Edit the `<version>`, `<description>`, `<maintainer>`, and `<license>` tags. Ensure that the version value is the same as the version string in `setup.py`.
- **[Optional]** Include other ROS2 dependencies. See https://ros.org/reps/rep-0149.html#dependency-tags. Use the `<depend>` tag if unsure how to include a dependency.

```xml
<?xml version="1.0"?>
<?xml-model href="http://download.ros.org/schema/package_format3.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>
<package format="3">
 <name>py_pkg_a</name>
  <version>0.0.0</version>
  <description>TODO: Package description</description>
  <maintainer email="todo@todo.com">TODO</maintainer>
  <license>TODO: License declaration</license>

  <depend>rclpy</depend>

  <export>
    <build_type>ament_python</build_type>
  </export>
</package>
```

## 3.3&emsp;`setup.cfg`
Declarative configuration file.
While using the template:
- Ensure that the file matches the contents.
- Substitute the 2x `py_pkg_a`.

```
[develop]
script_dir=$base/lib/py_pkg_a
[install]
install_scripts=$base/lib/py_pkg_a
```

## 3.4&emsp;`setup.py`
Distribution script.
While using the template:
- Ensure that the keyword arguments below (looks like `something=`) exist. There can be other keyword arguments.
- Substitute the 2x `py_pkg_a`, 1x `filename_a`, and 1x `executable_a`. These are used by the CLI command `ros2 run py_pkg_a executable_a` to run the executable created from the `filename_a.py` script. `executable_a` is typically the same as `filename_a`.
- **[Optional]** Edit the version, license, maintainer, maintainer email, and description. They should match the ones in `package.xml`, especially the version.
- **[Optional]** Include other ROS2 dependencies in `include=`.

```python
from setuptools import find_packages, setup

package_name = 'py_pkg_a'

setup(
    name=package_name,
    version='0.0.0',
    packages=find_packages(
        include=['rclpy'],
    ),
    data_files=[
        ('share/ament_index/resource_index/packages',
            ['resource/' + package_name]),
        ('share/' + package_name, ['package.xml']),
    ],
    install_requires=['setuptools'],
    zip_safe=True,
    maintainer='TODO',
    maintainer_email='todo@todo.com',
    description='TODO: Package description',
    license='TODO: License declaration',
    entry_points={
        'console_scripts': [
            'executable_a = py_pkg_a.filename_a:main'
        ],
    },
)
```

## 3.5&emsp;Multiple Nodes in a Package
This section details the structure for a Python package containing multiple scripts each implementing a node. 
`setup.py` must be changed to be able to run the new nodes.

- The directory structure for the Python scripts `filename_b.py` and `filename_c.py` that implement two new nodes are shown below. 
They must be placed in the same folder as all the other Python scripts that implement nodes.

    <table><tbody><tr><td>
        <details open>
            <summary><code>workspace_a/</code>&emsp;Workspace.</summary>
            <dl>
                <dd><details open> 
                    <summary><code>src/</code></summary>
                    <dl>
                        <dd><details open>
                            <summary><code>py_pkg_a/</code>&emsp;Python package.</summary>
                            <dl>
                                <dd><details open>
                                    <summary><code>py_pkg_a/</code></summary>
                                    <dl>
                                        <dd><code>__init__.py</code></dd>
                                        <dd><code>filename_a.py</code></dd>
                                        <dd><code>filename_b.py</code>&emsp;New node.</dd>
                                        <dd><code>filename_c.py</code>&emsp;New node.</dd>
                                    </dl>
                                </details></dd>
                                <dd><details>
                                    <summary><code>resource/</code></summary>
                                    <dl>
                                        <dd><code>py_pkg_a</code></dd>
                                    </dl>
                                </details></dd>
                                <dd><code>package.xml</code></dd>
                                <dd><code>setup.cfg</code></dd>
                                <dd><code>setup.py</code></dd>
                            </dl>
                        </details></dd>
                    </dl>
                </details></dd>
            </dl>
        </details>
    </td></tr></tbody></table>

- Edit the `entry_points` keyword argument in `setup.py` to account for the new scripts. Please pay attention to the commas separating the strings.
    ```python
    entry_points={
        'console_scripts': [
            'executable_a = py_pkg_a.filename_a:main',
            'executable_b = py_pkg_a.filename_b:main',
            'executable_c = py_pkg_a.filename_c:main'
        ]
    }
    ```

## 3.6&emsp;Multiple Packages

The directory implementing a new Python package should be placed within the `src` folder. In the example below, two new Python packages `py_pkg_b` and `py_pkg_c` are introduced.

Like `py_pkg_a` in the previous sections, ensure that all the files in the new packages `py_pkg_b` and `py_pkg_c` are edited properly.

The directory structure below shows the minimal structure of the three packages:
<table><tbody><tr><td>
    <details open>
    <summary><code>workspace_a/</code>&emsp;Workspace.</summary>
    <dl>
        <dd><details open> 
            <summary><code>src/</code></summary>
            <dl>
                <dd><details>
                    <summary><code>py_pkg_a/</code>&emsp;Python package 1.</summary>
                    <dl>
                        <dd><details open>
                            <summary><code>py_pkg_a/</code></summary>
                            <dl>
                                <dd><code>__init__.py</code></dd>
                                <dd><code>filename_a.py</code></dd>
                            </dl>
                        </details></dd>
                        <dd><details open>
                            <summary><code>resource/</code></summary>
                            <dl>
                                <dd><code>py_pkg_a</code></dd>
                            </dl>
                        </details></dd>
                        <dd><code>package.xml</code></dd>
                        <dd><code>setup.cfg</code></dd>
                        <dd><code>setup.py</code></dd>
                    </dl>
                </details></dd>
                <dd><details>
                    <summary><code>py_pkg_b/</code>&emsp;Python package 2.</summary>
                    <dl>
                        <dd><details open>
                            <summary><code>py_pkg_b/</code></summary>
                            <dl>
                                <dd><code>__init__.py</code></dd>
                                <dd><code>filename_a.py</code>&emsp;Can be the same or different file name.</dd>
                            </dl>
                        </details></dd>
                        <dd><details open>
                            <summary><code>resource/</code></summary>
                            <dl>
                                <dd><code>py_pkg_b</code></dd>
                            </dl>
                        </details></dd>
                        <dd><code>package.xml</code></dd>
                        <dd><code>setup.cfg</code></dd>
                        <dd><code>setup.py</code></dd>
                    </dl>
                </details></dd>
                <dd><details>
                    <summary><code>py_pkg_c/</code>&emsp;Python package 3.</summary>
                    <dl>
                        <dd><details open>
                            <summary><code>py_pkg_c/</code></summary>
                            <dl>
                                <dd><code>__init__.py</code></dd>
                                <dd><code>filename_a.py</code>&emsp;Can be the same or different file name.</dd>
                            </dl>
                        </details></dd>
                        <dd><details open>
                            <summary><code>resource/</code></summary>
                            <dl>
                                <dd><code>py_pkg_c</code></dd>
                            </dl>
                        </details></dd>
                        <dd><code>package.xml</code></dd>
                        <dd><code>setup.cfg</code></dd>
                        <dd><code>setup.py</code></dd>
                    </dl>
                </details></dd>
            </dl>
        </details></dd>
    </dl>
    </details>
</td></tr></tbody></table>

# 4&emsp;Building and Running a Project
When a project is built, executables are generated for ROS2 to run.

## 4.1&emsp;Normal Build
By substituting `workspace_a`, run the following `colcon` command in the workspace to build the project.
The `build`, and `install` and `log` folders will be generated. The folders contain the built executables and their required scripts.

```bash
cd workspace_a

colcon build --symlink-install
```
The directory, after building, will look like:
<table><tbody><tr><td>
    <details open>
        <summary><code>workspace_a/</code>&emsp;</summary>
        <dl>
            <dd><details> 
                <summary><code>build/</code>&emsp;Generated by colcon.</summary>
                <dl><dd>...</dd></dl>
            </details></dd>
            <dd><details> 
                <summary><code>src/</code>&emsp;Source code.</summary>
                <dl><dd>...</dd></dl>
            </details></dd>
            <dd><details> 
                <summary><code>install/</code>&emsp;Generated by colcon.</summary>
                <dl><dd>...</dd></dl>
            </details></dd>
            <dd><details> 
                <summary><code>log/</code>&emsp;Generated by colcon.</summary>
                <dl><dd>...</dd></dl>
            </details></dd>
        </dl>
    </details>
</td></tr></tbody></table>

## 4.2&emsp;Clean Build
`colcon` makes incremental builds to the project depending on what files were changed since the last build. 
This makes building more efficient, but can cause problems when attempting to run the newly built executables.
Such problems can occur when the directory structure in `src` is changed extensively.

If you feel that the project is not building correctly, remove the `build`, `install`, `log` folders and rebuild the workspace:

```bash
cd workspace_a

rm -rf build install log

colcon build --symlink-install
```
If there are no changes to the behavior of the executables, then something is wrong with the code.

## 4.3&emsp;Re-building

Re-building a package is necessary when the directory structure and dependencies used for generating the executables change. 

To be specific, rebuilding is necessary when:
- Any node or package is renamed or created.
- `setup.py`, `setup.pkg`, and `package.xml` are modified.
- **[Not in this course]** New files are created in directories to be installed. If `--symlink-install` is not provided, then rebuild is necessary even when files are modified in these directories.
- **[Not in this course]** C++ files are modified.

Rebuilding is not necessary when:
- An existing Python script implementing a node is modified, and this script is present in the last built.
- **[Not in this course]** Any file or folder, not used for building the executables, are modified or created.

To rebuild, simply follow the instructions to build normally or build cleanly.

## 4.4&emsp;Running After Building

To run a node from a project, the environment in the current terminal needs to the know that the project exists. To do so, source the `setup.bash` generated by the build process:

```bash
cd workspace_a

source install/setup.bash
```

The `source` command only needs to be run once when:
- A new terminal is opened, and the terminal will run an executable from the project.
- A clean build is performed.

The ROS2 CLI command `ros2 run py_pkg_a executable_a` can then be used to run executables from this project.

# 5&emsp;Moving Projects

It is common for a project to be moved between computers or directories for development and testing. 
When moving a project, **only the source code should be moved**. 

All things that can be built or generated from the source code such as log files or executables **should not be moved**. 
This is because the executables cannot be run from another location, and the executables (~MB) are typically much larger than the source code (~KB). 

This means that the `build`, `log`, and `install` folders should not be transferred. If using Visual Studio Code with the Robotics Development Environment extension, the `.vscode` folder should also not be transferred.

## 5.1&emsp;Keep Only `src`

Since the `src` folder contains the source code, only the contents in the `src` folder need to be kept.

You can then do either of the following to transfer the project:

<table><tbody>
    <tr>
        <th>Level</th>
        <th>Description</th>
    </tr>
    <tr>
        <td><b>Workspace</b></td>
        <td>Zip the <code>workspace_a</code> directory minus the <code>build</code>, <code>log</code>, <code>install</code>, and <code>.vscode</code> folders. There may be other files like <code>.gitignore</code> or read-me files that you would like to transfer with the <code>src</code> folder. This approach is recommended for submission in this course.</td>
    </tr>
    <tr>
        <td><b>Package</b></td>
        <td>Zip all the package folders in the <code>src</code> folder so that the top level of the zip file contains all the package folders instead of <code>src</code>. This approach is used by developers to distribute their packages.</td>
    </tr>
</tbody></table>

## 5.2&emsp;`.gitignore` for GitHub

Uploading the project onto GitHub is just moving the project online. 
Only the source code should be kept. 
This section assumes that you have gone through [05_Private_GitHub_Repository.md](https://github.com/LaiYanKai/Misc/blob/main/rb2301/05_Private_GitHub_Repository.md). 

The hidden file `.gitignore` is used to ignore folders and files from the commit.
In our case, the `build`, `log`, `install` and `.vscode` folders must be ignored.

To ignore the folders, create a `.gitignore` file in the `workspace_a` directory.
Then, insert the following into the `.gitignore` file:
```
build/
log/
install/
.vscode/
```

Normally, the commit process ignores hidden files like `.gitignore`. However, VSCode automatically adds all hidden folders and files. 
With the `.gitignore`, starting from the next commit from VSCode, any updates to the four folders will be ignored.

# 6&emsp;Tasks

1. Make sure that the instructions on [05_Private_GitHub_Repository.md](https://github.com/LaiYanKai/Misc/blob/main/rb2301/05_Private_GitHub_Repository.md) were followed. You should have the workspace directory `workspace_a` as `~/rb2301`. Create the `src` folder if it does not exist.

2. Using the ROS2 CLI, create a Python package `rb2301_tutorial` containing the script `tutorial.py` that will be used to implement a node.

3. By editing the `setup.py` file, ensure that the executable built from `tutorial.py` can be run with `ros2 run rb2301_tutorial tut`. A normal build can be performed to test the command.

4. Create a new script `fake.py` that will be used to implement another node. The command `ros2 run rb2301_tutorial fake` must be able to run the executable built from this script. A normal build can be performed to test the command.

5. Insert a `.gitignore file` that ignores the `.vscode`, `build`, `install`, and `log` folders in the workspace.

6. Using VSCode, push the changes onto your private `rb2301` GitHub repository.

7. **[Q1]** Zip the project on the workspace level and submit it to the Canvas quiz.