# Python 控制系统仿真与 gtest Mock 数据生成

## 1. Python 控制系统仿真工具包

### 核心库

#### python-control（最推荐）
最接近 MATLAB Control System Toolbox 的 Python 库：

```bash
pip install control numpy scipy matplotlib
```

主要功能：
- 传递函数建模 (`tf`)
- 状态空间模型 (`ss`)
- 频域分析（Bode图、Nyquist图）
- 时域分析（阶跃响应、脉冲响应）
- 根轨迹、极点配置
- PID 控制器设计

```python
import control as ct

# 创建传递函数 G(s) = 1/(s^2 + 2s + 1)
sys = ct.tf([1], [1, 2, 1])

# 阶跃响应
t, y = ct.step_response(sys)

# Bode图
ct.bode_plot(sys)
```

#### SciPy.signal
内置于 SciPy 中，适合基础的系统分析：

```python
from scipy import signal

sys = signal.TransferFunction([1], [1, 2, 1])
t, y = signal.step(sys)
```

### 扩展工具

| 库名 | 用途 |
|------|------|
| Slycot | python-control 的可选依赖，提供更多数值算法 |
| CVXPY | 凸优化，适合最优控制问题 |
| SimPy | 离散事件仿真 |
| PyDy | 多体动力学仿真 |
| Assimulo | 微分方程求解器（ODE/DAE） |

### 与 MATLAB 对比

| 功能 | MATLAB | Python (control) |
|------|--------|------------------|
| 传递函数 | `tf(num, den)` | `ct.tf(num, den)` |
| 状态空间 | `ss(A,B,C,D)` | `ct.ss(A,B,C,D)` |
| 阶跃响应 | `step(sys)` | `ct.step_response(sys)` |
| Bode图 | `bode(sys)` | `ct.bode_plot(sys)` |

---

## 2. 用于 gtest Mock 数据生成的方案

### 方案对比

| 方案 | 复杂度 | 适用场景 |
|------|--------|----------|
| 生成静态数据文件 (CSV/JSON) | ⭐ | 简单场景，数据量适中 |
| 生成 C++ 头文件 | ⭐⭐ | 数据需要编译时确定 |
| 构建系统集成 (CMake) | ⭐⭐⭐ | 自动化构建流程 |
| pybind11 实时调用 | ⭐⭐⭐⭐ | 需要动态仿真 |

---

### 方案1：Python 建模 + 生成数据文件（推荐）

#### Python 端：被控对象建模与数据生成

```python
# plant_model.py
import numpy as np
import control as ct
import json
import csv

class PlantModel:
    """被控对象模型 - 以二阶系统为例"""
    
    def __init__(self, wn=10.0, zeta=0.7):
        """
        wn: 自然频率
        zeta: 阻尼比
        """
        num = [wn**2]
        den = [1, 2*zeta*wn, wn**2]
        self.sys = ct.tf(num, den)
        self.ss = ct.tf2ss(self.sys)
        
    def step_response(self, t_end=2.0, dt=0.001):
        """生成阶跃响应数据"""
        t = np.arange(0, t_end, dt)
        t_out, y_out = ct.step_response(self.sys, t)
        return t_out, y_out
    
    def simulate(self, t, u):
        """给定输入信号，仿真输出"""
        t_out, y_out = ct.forced_response(self.sys, t, u)
        return t_out, y_out
    
    def discretize(self, dt):
        """离散化模型"""
        return ct.sample_system(self.ss, dt, method='zoh')


def generate_mock_data(output_dir="."):
    """生成 gtest 需要的 mock 数据"""
    
    plant = PlantModel(wn=10.0, zeta=0.7)
    
    # 阶跃响应数据
    t_step, y_step = plant.step_response(t_end=1.0, dt=0.001)
    
    # 正弦输入响应
    t_sin = np.arange(0, 2.0, 0.001)
    u_sin = np.sin(2 * np.pi * 1.0 * t_sin)
    _, y_sin = plant.simulate(t_sin, u_sin)
    
    # 导出为 JSON
    mock_data = {
        "plant_params": {"wn": 10.0, "zeta": 0.7},
        "sample_time": 0.001,
        "step_response": {
            "time": t_step.tolist(),
            "output": y_step.tolist()
        },
        "sine_response": {
            "time": t_sin.tolist(),
            "input": u_sin.tolist(),
            "output": y_sin.tolist()
        }
    }
    
    with open(f"{output_dir}/plant_mock_data.json", 'w') as f:
        json.dump(mock_data, f, indent=2)
    
    print(f"Mock data generated in {output_dir}/")


if __name__ == "__main__":
    generate_mock_data("./test_data")
```

#### C++ gtest 端：读取 mock 数据

```cpp
// test_controller.cpp
#include <gtest/gtest.h>
#include <fstream>
#include <nlohmann/json.hpp>
#include <vector>

using json = nlohmann::json;

class PlantMockData {
public:
    struct Response {
        std::vector<double> time;
        std::vector<double> input;
        std::vector<double> output;
    };
    
    double wn, zeta, sample_time;
    Response step_response;
    Response sine_response;
    
    static PlantMockData LoadFromJson(const std::string& filepath) {
        std::ifstream f(filepath);
        json data = json::parse(f);
        
        PlantMockData mock;
        mock.wn = data["plant_params"]["wn"];
        mock.zeta = data["plant_params"]["zeta"];
        mock.sample_time = data["sample_time"];
        
        mock.step_response.time = 
            data["step_response"]["time"].get<std::vector<double>>();
        mock.step_response.output = 
            data["step_response"]["output"].get<std::vector<double>>();
        
        return mock;
    }
};

class ControllerTest : public ::testing::Test {
protected:
    PlantMockData mock_data;
    
    void SetUp() override {
        mock_data = PlantMockData::LoadFromJson("test_data/plant_mock_data.json");
    }
};

TEST_F(ControllerTest, StepResponseTracking) {
    MyController controller(/* params */);
    const auto& expected = mock_data.step_response;
    
    for (size_t i = 0; i < expected.time.size(); ++i) {
        double plant_output = expected.output[i];
        double control_signal = controller.compute(1.0, plant_output);
        // 验证控制器行为...
    }
}
```

---

### 方案2：生成 C++ 头文件

```python
# generate_cpp_header.py
import numpy as np
import control as ct

def generate_cpp_header(output_file="plant_mock_data.h"):
    wn, zeta = 10.0, 0.7
    sys = ct.tf([wn**2], [1, 2*zeta*wn, wn**2])
    
    t = np.arange(0, 1.0, 0.01)
    _, y = ct.step_response(sys, t)
    
    with open(output_file, 'w') as f:
        f.write("#pragma once\n")
        f.write("#include <array>\n\n")
        f.write("namespace PlantMock {\n\n")
        
        f.write(f"constexpr double kWn = {wn};\n")
        f.write(f"constexpr double kZeta = {zeta};\n")
        f.write(f"constexpr size_t kDataSize = {len(t)};\n\n")
        
        f.write(f"constexpr std::array<double, {len(t)}> kTime = {{\n    ")
        f.write(", ".join(f"{x:.4f}" for x in t))
        f.write("\n};\n\n")
        
        f.write(f"constexpr std::array<double, {len(y)}> kStepResponse = {{\n    ")
        f.write(", ".join(f"{x:.6f}" for x in y))
        f.write("\n};\n\n")
        
        f.write("}  // namespace PlantMock\n")

if __name__ == "__main__":
    generate_cpp_header()
```

---

### 方案3：CMake 集成

```cmake
# CMakeLists.txt
find_package(Python3 REQUIRED)

add_custom_command(
    OUTPUT ${CMAKE_BINARY_DIR}/test_data/plant_mock_data.json
    COMMAND ${Python3_EXECUTABLE} ${CMAKE_SOURCE_DIR}/scripts/plant_model.py
    DEPENDS ${CMAKE_SOURCE_DIR}/scripts/plant_model.py
    COMMENT "Generating plant mock data..."
)

add_custom_target(generate_mock_data ALL
    DEPENDS ${CMAKE_BINARY_DIR}/test_data/plant_mock_data.json
)

add_executable(controller_test test_controller.cpp)
add_dependencies(controller_test generate_mock_data)
target_link_libraries(controller_test gtest gtest_main)
```

---

### 方案4：pybind11 实时调用

#### 原理
pybind11 允许 C++ 嵌入 Python 解释器，在 gtest 运行时实时调用 Python 模型。

```
┌─────────────────────────────────────────────────────────┐
│                    C++ gtest 进程                        │
│  ┌─────────────┐     ┌──────────────────────────────┐  │
│  │  测试用例    │────▶│  pybind11 嵌入式 Python     │  │
│  │             │◀────│  解释器                      │  │
│  └─────────────┘     └──────────────────────────────┘  │
│                                 │                       │
│                                 ▼                       │
│                        ┌─────────────────┐              │
│                        │  plant_model.py │              │
│                        └─────────────────┘              │
└─────────────────────────────────────────────────────────┘
```

#### Python 仿真器

```python
# plant_model.py
import numpy as np
import control as ct

class PlantSimulator:
    def __init__(self, wn=10.0, zeta=0.7, dt=0.001):
        self.dt = dt
        num = [wn**2]
        den = [1, 2*zeta*wn, wn**2]
        sys_c = ct.tf(num, den)
        ss_c = ct.tf2ss(sys_c)
        ss_d = ct.sample_system(ss_c, dt, method='zoh')
        
        self.A = np.array(ss_d.A)
        self.B = np.array(ss_d.B)
        self.C = np.array(ss_d.C)
        self.D = np.array(ss_d.D)
        self.x = np.zeros((self.A.shape[0], 1))
        
    def reset(self):
        self.x = np.zeros((self.A.shape[0], 1))
        
    def step(self, u: float) -> float:
        u = np.array([[u]])
        y = self.C @ self.x + self.D @ u
        self.x = self.A @ self.x + self.B @ u
        return float(y[0, 0])

# 全局仿真器接口
_simulator = None

def init_plant(wn=10.0, zeta=0.7, dt=0.001):
    global _simulator
    _simulator = PlantSimulator(wn, zeta, dt)

def plant_step(u: float) -> float:
    return _simulator.step(u)

def plant_reset():
    _simulator.reset()
```

#### C++ 测试代码

```cpp
// test_controller_realtime.cpp
#include <gtest/gtest.h>
#include <pybind11/embed.h>
#include <pybind11/stl.h>

namespace py = pybind11;

class RealtimePlantTest : public ::testing::Test {
protected:
    py::scoped_interpreter guard_{};
    py::module_ plant_module_;
    
    void SetUp() override {
        py::module_::import("sys").attr("path").attr("append")("./scripts");
        plant_module_ = py::module_::import("plant_model");
        plant_module_.attr("init_plant")(10.0, 0.7, 0.001);
    }
    
    void TearDown() override {
        plant_module_.attr("plant_reset")();
    }
    
    double PlantStep(double u) {
        return plant_module_.attr("plant_step")(u).cast<double>();
    }
};

TEST_F(RealtimePlantTest, StepResponse) {
    std::vector<double> response;
    for (int i = 0; i < 1000; ++i) {
        double y = PlantStep(1.0);
        response.push_back(y);
    }
    EXPECT_NEAR(response.back(), 1.0, 0.01);
}

TEST_F(RealtimePlantTest, PIDClosedLoop) {
    double kp = 2.0, ki = 5.0, kd = 0.1;
    double integral = 0.0, prev_error = 0.0, dt = 0.001;
    double setpoint = 1.0, y = 0.0;
    
    for (int i = 0; i < 2000; ++i) {
        double error = setpoint - y;
        integral += error * dt;
        double derivative = (error - prev_error) / dt;
        double u = kp * error + ki * integral + kd * derivative;
        y = PlantStep(u);
        prev_error = error;
    }
    EXPECT_NEAR(y, setpoint, 0.01);
}
```

#### CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.15)
project(controller_test)

set(CMAKE_CXX_STANDARD 17)

find_package(Python3 COMPONENTS Interpreter Development REQUIRED)
find_package(pybind11 CONFIG REQUIRED)
find_package(GTest REQUIRED)

add_executable(realtime_plant_test test_controller_realtime.cpp)

target_link_libraries(realtime_plant_test
    PRIVATE
        pybind11::embed
        GTest::gtest
        GTest::gtest_main
)
```

#### pybind11 优缺点

| ✅ 优点 | ❌ 缺点 |
|---------|---------|
| 极高灵活性 | 需要 Python 运行时环境 |
| 无需预生成数据 | 配置较复杂 |
| 可动态修改参数 | 性能开销 |
| 模型与测试同步更新 | CI/CD 需配置 Python |

---

## 3. 推荐的项目结构

```
project/
├── CMakeLists.txt
├── scripts/
│   ├── plant_model.py          # Python 被控对象模型
│   └── generate_mock_data.py   # 数据生成脚本
├── test_data/                  # 生成的 mock 数据
│   ├── plant_mock_data.json
│   └── step_response.csv
├── src/
│   └── controller.cpp          # 控制器实现
└── test/
    ├── test_controller.cpp     # gtest 测试
    └── mock_data_loader.h      # 数据加载工具
```

---

## 4. 总结建议

1. **开发阶段**：用 pybind11 实时调用 → 快速迭代
2. **回归测试**：用静态数据文件 (JSON) → 稳定可靠
3. **CI/CD**：构建系统集成 (CMake) → 自动化
4. 可以两者结合：用 pybind11 生成"黄金数据"，然后保存为 JSON 用于 CI

