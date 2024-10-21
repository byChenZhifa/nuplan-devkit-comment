该代码定义了一个名为 `SimulationRunner` 的类，这个类用于管理和执行多个模拟任务。下面我将详细解释代码中的各个部分。

### 1. 引入模块

```python
from __future__ import annotations

import logging
import time
from typing import Any, Callable, List

from nuplan.planning.scenario_builder.abstract_scenario import AbstractScenario
from nuplan.planning.simulation.planner.abstract_planner import AbstractPlanner
from nuplan.planning.simulation.runner.abstract_runner import AbstractRunner
from nuplan.planning.simulation.runner.runner_report import RunnerReport
from nuplan.planning.simulation.simulation import Simulation
```

- `__future__ import annotations`: 这个语句使得在 Python 3.7+ 中可以使用延迟的类型注解。这样的话，类型注解中的类名可以使用字符串而无需在实际定义之前导入。
- `logging`: 用于记录日志信息。
- `time`: 提供时间测量功能，这里用于计算模拟运行的时间。
- `typing`: 提供了类型注解，如 `Any`, `Callable`, `List` 等，用于定义函数参数和返回值的类型。
- 其他导入是从 `nuplan` 框架中引入的类，用于管理和执行模拟任务。

### 2. `for_each` 函数

```python
def for_each(fn: Callable[[Any], Any], items: List[Any]) -> None:
    """
    Call function on every item in items
    :param fn: function to be called fn(item)
    :param items: list of items
    """
    for item in items:
        fn(item)
```

这个函数接收一个函数 `fn` 和一个列表 `items`，然后对列表中的每个元素调用 `fn` 函数。这个模式通常用于对一组对象执行统一的操作。

### 3. `SimulationRunner` 类

```python
class SimulationRunner(AbstractRunner):
    """
    Manager which executes multiple simulations with the same planner
    """
```

- `SimulationRunner` 继承自 `AbstractRunner`，用于管理和执行多个模拟任务。

#### 初始化方法

```python
def __init__(self, simulation: Simulation, planner: AbstractPlanner):
    """
    Initialize the simulations manager
    :param simulation: Simulation which will be executed
    :param planner: which should be used to compute the desired ego's trajectory
    """
    self._simulation = simulation
    self._planner = planner
```

- `__init__` 方法用于初始化 `SimulationRunner` 对象。它接受一个 `Simulation` 对象和一个 `AbstractPlanner` 对象，并将它们存储在实例变量 `_simulation` 和 `_planner` 中。

#### 内部初始化方法

```python
def _initialize(self) -> None:
    """
    Initialize the planner
    """
    # Execute specific callback
    self._simulation.callback.on_initialization_start(self._simulation.setup, self._planner)

    # Initialize Planner
    self.planner.initialize(self._simulation.initialize())

    # Execute specific callback
    self._simulation.callback.on_initialization_end(self._simulation.setup, self._planner)
```

- `_initialize` 方法用于初始化模拟任务和规划器。调用了一系列的回调函数，确保在初始化开始和结束时执行特定的操作。

#### 属性方法

```python
@property
def planner(self) -> AbstractPlanner:
    """
    :return: Planner used by the SimulationRunner
    """
    return self._planner

@property
def simulation(self) -> Simulation:
    """
    :return: Simulation used by the SimulationRunner
    """
    return self._simulation

@property
def scenario(self) -> AbstractScenario:
    """
    :return: Get the scenario relative to the simulation.
    """
    return self.simulation.scenario
```

- 这些 `@property` 方法用于访问 `SimulationRunner` 的属性，分别返回规划器、模拟任务和场景。

#### 运行方法

```python
def run(self) -> RunnerReport:
    """
    Run through all simulations. The steps of execution follow:
     - Initialize all planners
     - Step through simulations until there no running simulation
    :return: List of SimulationReports containing the results of each simulation
    """
    start_time = time.perf_counter()

    # Initialize reports for all the simulations that will run
    report = RunnerReport(
        succeeded=True,
        error_message=None,
        start_time=start_time,
        end_time=None,
        planner_report=None,
        scenario_name=self._simulation.scenario.scenario_name,
        planner_name=self.planner.name(),
        log_name=self._simulation.scenario.log_name,
    )

    # Execute specific callback
    self.simulation.callback.on_simulation_start(self.simulation.setup)

    # Initialize all simulations
    self._initialize()

    while self.simulation.is_simulation_running():
        # Execute specific callback
        self.simulation.callback.on_step_start(self.simulation.setup, self._planner)

        # Perform step
        planner_input = self._simulation.get_planner_input()
        logger.debug("Simulation iterations: %s" % planner_input.iteration.index)

        # Execute specific callback
        self._simulation.callback.on_planner_start(self.simulation.setup, self.planner)

        # Plan path based on all planner's inputs
        trajectory = self.planner.compute_trajectory(planner_input)

        # Propagate simulation based on planner trajectory
        self._simulation.callback.on_planner_end(self.simulation.setup, self.planner, trajectory)
        self.simulation.propagate(trajectory)

        # Execute specific callback
        self.simulation.callback.on_step_end(self.simulation.setup, self.planner, self.simulation.history.last())

        # Store reports for simulations which just finished running
        current_time = time.perf_counter()
        if not self.simulation.is_simulation_running():
            report.end_time = current_time

    # Execute specific callback
    self.simulation.callback.on_simulation_end(self.simulation.setup, self.planner, self.simulation.history)

    planner_report = self.planner.generate_planner_report()
    report.planner_report = planner_report

    return report
```

- `run` 方法是 `SimulationRunner` 的核心，它管理了模拟任务的整个执行流程。具体步骤包括初始化规划器、迭代执行模拟任务的每一步操作，并在模拟结束后生成并返回一个 `RunnerReport` 对象。整个过程中使用了一些回调函数来执行特定的操作，同时利用 `time.perf_counter` 来测量执行时间。



# 解析


`SimulationRunner` 类的 `run` 函数是整个模拟执行流程的核心部分，它管理模拟任务的初始化、执行和收尾工作。我们来详细解析 `run` 函数的执行过程，并结合 `Simulation` 类中的方法，理解它如何协调模拟运行。

### 1. 启动计时器

```python
start_time = time.perf_counter()
```
- 使用 `time.perf_counter()` 来记录模拟开始的时间，以便之后计算总的执行时间。

### 2. 初始化 `RunnerReport`

```python
report = RunnerReport(
    succeeded=True,
    error_message=None,
    start_time=start_time,
    end_time=None,
    planner_report=None,
    scenario_name=self._simulation.scenario.scenario_name,
    planner_name=self.planner.name(),
    log_name=self._simulation.scenario.log_name,
)
```
- 初始化 `RunnerReport` 对象，用于记录模拟的结果和元数据。初始状态下，`succeeded` 标志设置为 `True`，`error_message` 为 `None`。还记录了模拟开始的时间、场景名称、规划器名称等。

### 3. 调用回调函数：模拟开始

```python
self.simulation.callback.on_simulation_start(self.simulation.setup)
```
- 调用 `on_simulation_start` 回调函数，通知系统模拟已开始。这通常用于记录日志或执行其他启动时的操作。

### 4. 初始化所有模拟

```python
self._initialize()
```
- 调用 `_initialize` 方法，初始化规划器和模拟。该方法会触发一系列回调函数，包括 `on_initialization_start` 和 `on_initialization_end`，并通过调用 `Simulation.initialize()` 来重置模拟状态、初始化历史缓冲区、加载初始观测数据等。

### 5. 进入模拟执行循环

```python
while self.simulation.is_simulation_running():
```
- `is_simulation_running` 方法检查模拟是否仍在运行。只要没有到达时间控制器的结束条件，模拟就会继续。

### 6. 每一步模拟执行

#### 6.1 步骤开始回调

```python
self.simulation.callback.on_step_start(self.simulation.setup, self._planner)
```
- 每次模拟迭代开始时，触发 `on_step_start` 回调函数。这通常用于执行某些在每个时间步开始时所需的操作，比如记录日志。

#### 6.2 获取规划器输入

```python
planner_input = self._simulation.get_planner_input()
logger.debug("Simulation iterations: %s" % planner_input.iteration.index)
```
- 调用 `Simulation.get_planner_input()` 获取当前时间步的规划器输入，包括当前状态、历史数据、交通信号状态等。然后记录当前迭代的索引值。

#### 6.3 规划器开始回调

```python
self._simulation.callback.on_planner_start(self.simulation.setup, self.planner)
```
- 在调用规划器进行路径规划之前，触发 `on_planner_start` 回调函数。这通常用于标记规划器的开始，以便后续可能的性能分析或日志记录。

#### 6.4 计算规划器的轨迹

```python
trajectory = self.planner.compute_trajectory(planner_input)
```
- 使用 `planner_input` 调用规划器的 `compute_trajectory` 方法，计算出当前时间步内理想的车辆轨迹。

#### 6.5 规划器结束回调

```python
self._simulation.callback.on_planner_end(self.simulation.setup, self.planner, trajectory)
```
- 规划器完成计算后，触发 `on_planner_end` 回调函数。这通常用于记录规划器的输出或执行其他后续操作。

#### 6.6 传播模拟状态

```python
self.simulation.propagate(trajectory)
```
- 调用 `Simulation.propagate()` 方法，根据规划器的输出轨迹更新车辆状态，并决定模拟是否继续。此方法会更新自车状态、观测数据，并将新状态添加到历史缓冲区。

#### 6.7 步骤结束回调

```python
self.simulation.callback.on_step_end(self.simulation.setup, self.planner, self.simulation.history.last())
```
- 触发 `on_step_end` 回调函数，表示当前时间步已完成。通常会传递当前步骤的最终状态，以便在日志中记录或用于进一步分析。

### 7. 检查模拟是否结束

```python
current_time = time.perf_counter()
if not self.simulation.is_simulation_running():
    report.end_time = current_time
```
- 如果模拟已结束，记录结束时间到 `RunnerReport` 的 `end_time` 中。此时，模拟任务的主要循环已终止。

### 8. 调用回调函数：模拟结束

```python
self.simulation.callback.on_simulation_end(self.simulation.setup, self.planner, self.simulation.history)
```
- 在模拟完全结束后，触发 `on_simulation_end` 回调函数。这通常用于执行一些清理操作或最终的数据记录。

### 9. 生成规划器报告

```python
planner_report = self.planner.generate_planner_report()
report.planner_report = planner_report
```
- 在模拟结束后，生成并记录规划器的报告，包含规划器的性能统计数据等。

### 10. 返回 `RunnerReport`

```python
return report
```
- 最终，`run` 函数返回 `RunnerReport`，该报告包含模拟的完整结果和元数据。

### 总结

`SimulationRunner` 类的 `run` 函数在 `Simulation` 对象的协助下，管理并执行了整个模拟过程。它通过一系列回调函数和主要的模拟逻辑，完成了模拟任务的初始化、迭代和终止。最终生成的 `RunnerReport` 包含模拟运行的所有重要信息，供后续分析和验证使用。