# 🎉 Python入门培训手册

欢迎大家参加 2025 年秀钟暑期实践课程！🌱

本教程由老师和助教团队 🧑‍🏫👩‍💻 精心编写，旨在帮助大家高效入门本次实践所需的编程基础知识。无论你是初学者还是有一定经验，都可以通过本教程快速学习，变得更加睿智 🛠️💡。

请大家按照本教程逐步操作。如果在学习或实践过程中遇到问题，欢迎随时在班级群或课上向老师、助教提问，我们会及时为大家解答与支持 🤝。

预祝大家实践学习顺利，收获满满！🚀✨

> 指导老师: 张小乐 & 助教团队: 蹇姚宇蝶  
> 2025年6月27日

## 一、Hello, World!

```python
print("Hello, world!")
```

* **print**: 输出函数，用于展示内容。
* **"Hello, world!"**: 字符串。用双引号括起来的文本就是 string 类型。

---

## 二、Python 基础知识详解

以下为 Python 编程的基础构成部分，适用于初学者建立完整语言理解体系。我们将按模块依次进行简单说明与示例：

---

## 2.1 Python 简介

Python 是一种解释型、面向对象、动态数据类型的高级程序设计语言。

它由 Guido van Rossum 于 1989 年底发明，第一个公开发行版发布于 1991 年。Python 的语法简洁、结构清晰，被广泛应用于人工智能、数据科学、Web 开发、自动化运维等领域。

Python 源代码遵循 GPL（GNU General Public License）协议，拥有庞大的开源社区和库生态。

**与其他语言的对比：**

* **Python vs C/C++**：C/C++ 为编译型语言，执行前需显式编译；Python 是解释型语言，可直接运行代码。
* **Python vs C++**：C++ 更注重性能与底层控制，语法复杂，适合嵌入式/系统级开发；Python 强调可读性和快速开发，适合算法验证和脚本开发。
* **Python vs Java**：两者均为跨平台语言，但 Python 更“动态”，无需强类型声明、代码更简洁，适合快速迭代。

Python 的“优雅、明确、简单”哲学深受开发者欢迎。

**迭代历史：**

- 官方宣布，2020 年 1 月 1 日， 停止 Python 2 的更新。
- Python 2.7 被确定为最后一个 Python 2.x 版本。
- Python 的 3.0 版本，常被称为 Python 3000，或简称 Py3k。相对于 Python 的早期版本，这是一个较大的升级。为了不带入过多的累赘，Python 3.0 在设计的时候没有考虑向下兼容。

---

## 2.2 Python 安装和环境搭建

Python 可通过官网下载（[https://www.python.org/）安装，建议选择](https://www.python.org/）安装，建议选择) Python 3.x 版本。
推荐使用集成环境如 VSCode、PyCharm、Jupyter Notebook 或 Anaconda。
具体的安装和环境配置见`Software_Tutorial.md`。

---

## 2.3 Python 基础语法

* 使用缩进表示代码块（建议使用 4 个空格）
* 注释使用 `#` 开头，函数/类可使用三引号文档字符串
* Python 区分大小写

```python
# 单行注释
''' 多行注释或函数说明 '''
print("Hello")
```

---

## 2.4 Python 标识符

* 第一个字符必须以字母（a-z, A-Z）或下划线 `_` 。
* 标识符的其他的部分由字母、数字和下划线组成。
* 标识符对大小写敏感，count 和 Count 是不同的标识符。
* 标识符对长度无硬性限制，但建议保持简洁（一般不超过 20 个字符）。
* 禁止使用保留关键字，如 if、for、class 等不能作为标识符。

合法标识符：
```python
age = 25
user_name = "Alice"
_total = 100
MAX_SIZE = 1024
calculate_area()
StudentInfo
__private_var
```

非法标识符：
```python
2nd_place = "silver"    # 错误：以数字开头
user-name = "Bob"       # 错误：包含连字符
class = "Math"          # 错误：使用关键字
$price = 9.99          # 错误：包含特殊字符
for = "loop"           # 错误：使用关键字
```

---

## 2.5 Python 变量类型

* 不需要预先声明类型，赋值时自动推导：

```python
x = 5        # int
y = 3.14     # float
z = "hi"     # str
flag = True  # bool
```

---

## 2.6 Python import 与 from...import

* 在 python 用 import 或者 from...import 来导入相应的模块。
* 将整个模块(somemodule)导入，格式为： `import somemodule`
* 从某个模块中导入某个函数,格式为： `from somemodule import somefunction`
* 从某个模块中导入多个函数,格式为： `from somemodule import firstfunc, secondfunc, thirdfunc`
* 将某个模块中的全部函数导入，格式为： `from somemodule import *`

---

---

## 2.7 Python 运算符

* 算术运算符：`+ - * / // % **`
* 比较运算符：`== != > >= < <=`
* 逻辑运算符：`and or not`
* 成员运算：`in`，`not in`

---

---

## 2.8 Python 条件语句

```python
x = 10
if x > 0:
    print("正数")
elif x == 0:
    print("零")
else:
    print("负数")
```

---

## 2.9 Python 循环语句（while / for）

```python
for i in range(3):
    print(i)

while x > 0:
    x -= 1
```

---

## 2.10 Python 循环嵌套与控制语句（break / continue / pass）

* **break**：提前结束循环
* **continue**：跳过当次循环
* **pass**：占位语句，不做任何操作

```python
for i in range(5):
    if i == 2:
        continue
    print(i)
```

---

---

## 2.11 Python Number（数字类型）

包括整数（int）、浮点数（float）、复数（complex）：

```python
x = 10
pi = 3.14159
z = 2 + 3j
```

---

## 2.12 Python 字符串

字符串可以通过索引、切片访问，也支持常用方法如 `split()`、`replace()` 等：

```python
s = "Hello World"
print(s[0], s[-1])
print(s.lower(), s.upper())
```

---

## 2.13 Python 列表（List）

```python
from datetime import datetime, timedelta
now = datetime.now()
tomorrow = now + timedelta(days=1)
print(now.strftime("%Y-%m-%d %H:%M:%S"))
```

---

---

## 2.14 Python 元组（Tuple）

不可变序列类型，适合用于多值返回：

```python
t = (1, 2, 3)
```

---

## 2.15 Python 字典（Dictionary）

键值对集合，常用于存储结构化信息：

```python
d = {"name": "Alice", "age": 25}
print(d["name"])
```

---

## 2.16 Python 日期和时间

```python
from datetime import datetime, timedelta
now = datetime.now()
print(now.strftime("%Y-%m-%d %H:%M:%S"))
```

---

## 2.17 Python 函数

```python
def greet(name):
    return "Hello, " + name
```

---

## 三、Python 实战能力介绍（库与工具）

在掌握基础语法之后，Python 强大的生命力来自于其丰富的第三方库生态。以下是常用于科学计算、数据分析、深度学习、可视化等实际项目中的库简介：

### 3.1 科学计算与数据分析

* **NumPy**：多维数组、高效矩阵运算，是大多数科学库的基础。
* **Pandas**：结构化数据分析工具，DataFrame 是数据处理核心对象。
* **SciPy**：包含优化、积分、插值、信号处理等高级数值计算。

### 3.2 数据可视化

* **Matplotlib**：最常用的画图工具，支持静态折线图、柱状图等。
* **Seaborn**：基于 Matplotlib 的统计图表库，风格美观。

### 3.3 深度学习与 AI

* **TensorFlow**：谷歌推出的深度学习框架，适合部署与工业级开发。
* **Keras**：TensorFlow 的高级接口，简化神经网络构建流程。
* **PyTorch**：Facebook 推出的框架，灵活、适合研究与实验。

### 3.4 Web 开发与爬虫

* **Flask**：轻量级 Web 框架，适合构建接口或快速原型。
* **Django**：全功能 Web 框架，支持 ORM、后台、用户系统等。
* **Scrapy**：网络爬虫框架，适合大规模网页采集。

### 3.5 计算机视觉与图像处理

* **OpenCV**：广泛用于图像识别、滤波、目标检测、摄像头处理等。

### 3.6 桌面 GUI 应用

* **PyQt / PySide**：基于 Qt 库的图形界面开发，功能丰富、跨平台。

> 在我们的课程中，这些库将作为基础能力介绍，重点仍会落在**我们自己的数据采集与分析项目代码**上，通过真实案例掌握 Python 编程在实际场景中的运用。

---

## 四、实战练习：我们的课程代码

在掌握以上基础与工具后，课程的重点将落在我们自己的传感器数据处理项目上，目标包括：

* 学会如何构建数据生成器与结构化存储格式（JSON/CSV）
* 掌握数据清洗与预处理（插值、异常剔除）
* 完成从原始数据到图表展示的全过程
* 理解多张图联动、滑动平均、直方图与双Y轴的可视化技巧

实战代码见`python_plot.md`。

---

参考：
- [runoob Python 教程](https://www.runoob.com/python3/python3-tutorial.html)
- [Python入门教程](https://zhuanlan.zhihu.com/p/1919062532817137924)