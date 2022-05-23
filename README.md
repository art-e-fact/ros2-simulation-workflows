# ROS2 Simulation workflows

Tips, scripts and examples of workflows using simulation in ROS2.
This is a **work in progress**. Contributions welcome.

## Integration tests

You can use [launch_testing](https://github.com/ros2/launch/tree/master/launch_testing) to define tests.

It allows you to define nodes running in a test similarly to what would be done in a launch file bu using `generate_test_description` instead of `generate_launch_description`.

```python
import unittest

import pytest
import launch_testing
from launch import LaunchDescription


@pytest.mark.launch_test
def generate_test_description():
    return LaunchDescription([
        Node(
            package="turtlesim",
            executable='turtlesim_node',
        ),
        launch_testing.actions.ReadyToTest()
    ])

@launch_testing.post_shutdown_test()
class TestProcessOutput(unittest.TestCase):
    def test_exit_code(self, proc_info):
        # Check that all processes in the launch (in this case, there's just one) exit
        # with code 0
        launch_testing.asserts.assertExitCodes(proc_info)
class TestProcessOutput(unittest.TestCase):
    def test_exit_code(self, proc_info):
        # Check that all processes in the launch (in this case, there's just one) exit
        # with code 0
        launch_testing.asserts.assertExitCodes(proc_info)
```

Running tests can be done with the `launch_test` command, for example:

```shell
launch_test launch_turtle_tests.py
```


### Integration with CI tools

You can export test results with the `--junit-xml` options, which create Junit compatible tests results, meaning it can be used in frameworks like [Jenkins](https://www.jenkins.io/),

```shell
launch_test launch_turtle_tests.py --junit-xml results.xml
```

The `results.xml` file can be parsed by continuous integration tools and contains description of `test suites` which map to the test launch file, and `test cases` which map to each individual test method inside unittest's `TestCase`.



### Creating tests

Initial conditions from live session

Best practice to define tests:
* Make sure your simulation is publishing ground truth information for what you want to test
* Create a node that uses topic subscriptions to estimate success
* Make sure that a test ends: using a timeout is usually the easiest way


## Parameter optimization

A common use case in simulation is to find an algorithm parameter  that optimized a desired metric.
Trial and error is typically used, but this is error prone and time consuming, especially if this is is to be done for multiple customer dpeloyments.


The strategy is to:

* define the range of parameters to test
* define a metric topic
* perform a grid search to run test on all parameter combinations and report the best performing set.

An example parameter using SMAC instead of grid search on a specific use case:
https://github.com/oscar-lima/autom_param_optimization

## References

* [ROS 2: What’s new?](https://static1.squarespace.com/static/51df34b1e4b08840dcfd2841/t/5e4e3ad992e2c26a1550ef53/1582185179043/2020-02-18_ROS+2+What%27s+new_JacobPerron_OpenRobotics.pdf)
* https://github.com/ros2/launch_ros/blob/master/launch_testing_ros/test/examples/talker_listener_launch_test.py
* https://discourse.ros.org/t/test-framework-in-ros2/2667/3
* https://answers.ros.org/question/356180/ros2-creating-integration-tests-for-python-nodes/
* https://answers.ros.org/question/377754/ros2foxylaunch-testing-integration-testing-with-launch-testing/
* https://autowarefoundation.gitlab.io/autoware.auto/AutowareAuto/integration-testing.html
* https://github.com/ros2/launch/issues/166
* https://github.com/ApexAI/apex_rostest
