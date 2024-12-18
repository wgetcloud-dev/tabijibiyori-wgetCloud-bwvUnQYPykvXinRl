[合集 \- 玩转Streamlit(15\)](https://github.com)[1\.什么是Streamlit10\-11](https://github.com/wang_yb/p/18458062)[2\.『玩转Streamlit』\-\-环境配置10\-17](https://github.com/wang_yb/p/18471660)[3\.『玩转Streamlit』\-\-架构和运行机制10\-22](https://github.com/wang_yb/p/18492213)[4\.『玩转Streamlit』\-\-多页应用10\-25](https://github.com/wang_yb/p/18502232)[5\.『玩转Streamlit』\-\-页面布局10\-31](https://github.com/wang_yb/p/18516928)[6\.『玩转Streamlit』\-\-登录认证机制11\-05](https://github.com/wang_yb/p/18527320)[7\.『玩转Streamlit』\-\-文本与标题组件11\-07](https://github.com/wang_yb/p/18531821):[FlowerCloud机场订阅官网](https://hanlianfangzhi.com)[8\.『玩转Streamlit』\-\-数据展示组件11\-13](https://github.com/wang_yb/p/18543687)[9\.『玩转Streamlit』\-\-图像与媒体组件11\-16](https://github.com/wang_yb/p/18549720)[10\.『玩转Streamlit』\-\-交互类组件11\-19](https://github.com/wang_yb/p/18554142)[11\.『玩转Streamlit』\-\-布局与容器组件11\-23](https://github.com/wang_yb/p/18564468)[12\.『玩转Streamlit』\-\-可编辑表格11\-29](https://github.com/wang_yb/p/18576592)[13\.『玩转Streamlit』\-\-表单Form12\-04](https://github.com/wang_yb/p/18585991)[14\.『玩转Streamlit』\-\-片段Fragments12\-10](https://github.com/wang_yb/p/18597017)15\.『玩转Streamlit』\-\-集成Matplotlib12\-17收起
`Steamlit`虽然也自带了一些绘图组件（比如折线图，柱状图和散点图等等），但是都比较简单，


和`Python`传统的可视化库比起来，功能上差了很多。


本篇介绍如何在`Streamlit App`中使用`Matplotlib`库来绘图。


# 1\. st.pyplot函数


`st.pyplot`函数专门用于在`Steamlit`应用中显示 `Matplotlib` 绘制的图形。


这个函数能够直接将`Matplotlib Figure`对象直接渲染到页面的指定位置上。


`st.pyplot`的参数不多，主要有：




| **名称** | **类型** | **说明** |
| --- | --- | --- |
| fig | Figure对象 | 要渲染的 Matplotlib Figure 对象 |
| clear\_figure | bool | 控制图形渲染后是否清除 |
| use\_container\_width | bool | 决定是否使用父容器的宽度覆盖图形的原始宽度 |


最重要的就是`fig`参数，它是通过 `Matplotlib` 的常规绘图方式创建图形对象。


也就是说，我们绘制图形时，完全不用考虑`Streamlit`，正常使用`Matplotlib`来绘图，


绘制之后直接将`Matplotlib`的`fig`对象传给`st.pyplot`函数即可。


# 2\. Matplotlib兼容性问题


在使用 `Matplotlib` 与 `Streamlit` 结合时，可能会遇到一些兼容性问题。


`Matplotlib` 支持多种 `Backend`，如果在使用过程中出现错误，可以尝试将`Backend`设置为`TkAgg`。


具体的设置方法如下：


1. 打开`~/.matplotlib/matplotlibrc`文件，如果不存在就创建一个
2. 在上面的文件中添加一行：`backend: TkAgg`


如果是windows系统，上面的文件路径改为：`C:\Users\%username%\.matplotlib\matplotlibrc`


此外，`Matplotlib` 在多线程环境下可能会出现问题，因为它本身对线程的支持并不完善。


当部署和共享应用程序时，由于可能存在并发用户，这个问题可能会更加突出。


为了解决这个问题，建议使用`RendererAgg.lock`来包裹 `Matplotlib` 相关代码，如下所示：



```
from matplotlib.backends.backend_agg import RendererAgg
_lock = RendererAgg.lock

with _lock:
    fig.title('sample figure')
    fig.plot([1,20,3,50])
    st.pyplot(fig)

```

# 3\. 使用示例


下面通过两个根据实际情况简化的示例来演示两者结合的效果。


## 3\.1\. 模拟数据分析项目


这个示例中，我们使用 `np.random` 函数生成了包含三列不同类型随机数据的 `DataFrame`。


在数据分析与可视化部分，根据选择的列进行不同类型的绘图操作，


* 当选择 `Column1` 或 `Column2` 时绘制柱状图或直方图
* 当选择 `Column3` 时可选择绘制 `Column1` 和 `Column2` 的散点图


本示例主要展示了 `Streamlit` 与 `Matplotlib` 结合在数据探索和分析可视化方面的便利性与交互性。



```
import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np

import matplotlib

# 为了显示中文
matplotlib.rcParams["font.sans-serif"] = ["Microsoft YaHei Mono"]
matplotlib.rcParams["axes.unicode_minus"] = False


# 生成随机数据
@st.cache_data
def generate_data():
    # 生成 100 行 3 列的随机数据
    data = {
        "Column1": np.random.randint(1, 100, 100),
        "Column2": np.random.normal(50, 10, 100),
        "Column3": np.random.choice(["A", "B", "C"], 100),
    }
    return pd.DataFrame(data)


data = generate_data()

# 数据探索部分
st.subheader("数据探索")
st.write(data.head())

# 数据分析与可视化
st.subheader("数据分析与可视化")
# 选择分析的列
selected_column = st.selectbox("选择要分析的列", data.columns)

# 使用 Matplotlib 绘制柱状图
if selected_column in ["Column1", "Column2"]:
    fig, ax = plt.subplots()
    if selected_column == "Column1":
        sns.countplot(data=data, x=selected_column, ax=ax)
    else:
        sns.histplot(data=data, x=selected_column, kde=True, ax=ax)
    ax.set_title(f"{selected_column} 分布情况")
    ax.set_xlabel(selected_column)
    ax.set_ylabel("数量")
    st.pyplot(fig)

# 绘制散点图（以 Column1 和 Column2 为例）
elif selected_column == "Column3":
    if st.checkbox("显示散点图"):
        fig2, ax2 = plt.subplots()
        ax2.scatter(data["Column1"], data["Column2"])
        ax2.set_title("Column1 与 Column2 的关系")
        ax2.set_xlabel("Column1")
        ax2.set_ylabel("Column2")
        st.pyplot(fig2)

```

运行效果：


![](https://img2024.cnblogs.com/blog/83005/202412/83005-20241217220232096-1419263696.gif)


## 3\.2\. 模拟数据监控与报告


在这个数据监控与报告应用示例中，通过`get_live_data` 函数模拟获取实时数据（实际应用中可替换为真实的数据获取逻辑）。


然后通过一个循环，不断获取新数据并合并到总的数据集中，再使用 `Matplotlib` 绘制折线图展示数据随时间的变化趋势。


这样就构建了一个简单的实时数据监控应用，在实际业务场景中，例如监控服务器性能指标、生产线上的关键数据等，可以让相关人员实时直观地了解数据的变化情况，及时发现异常并做出决策。



```
import streamlit as st
import matplotlib.pyplot as plt
import time
import random
import pandas as pd

import matplotlib

# 为了显示中文
matplotlib.rcParams["font.sans-serif"] = ["Microsoft YaHei Mono"]
matplotlib.rcParams["axes.unicode_minus"] = False


# 模拟获取实时数据的函数
def get_live_data():
    # 这里可以替换为真实的获取数据逻辑，比如从数据库或 API 获取
    new_data = pd.DataFrame(
        {
            "time": [time.strftime("%H:%M:%S")],
            "value": [random.randint(-10, 10)],
        }
    )
    return new_data


# 初始化数据
data = pd.DataFrame(columns=["time", "value"])

# 实时数据监控应用标题
st.title("实时数据监控")

# 创建一个占位符用于更新图表
chart_placeholder = st.empty()

while True:
    # 获取新数据
    new_data = get_live_data()
    # 合并新数据到总数据
    data = pd.concat([data, new_data], ignore_index=True)

    # 使用 Matplotlib 绘制折线图
    fig, ax = plt.subplots()
    ax.plot(data["time"], data["value"])
    ax.set_title("实时数据趋势")
    ax.set_xlabel("时间")
    ax.set_ylabel("数值")
    ax.set_xticklabels(data["time"], rotation=45)

    # 在 Streamlit 中更新图表
    chart_placeholder.pyplot(fig)

    # 每隔一段时间更新数据（这里设置为 2 秒）
    time.sleep(2)

```

运行效果：


![](https://img2024.cnblogs.com/blog/83005/202412/83005-20241217220232041-894469533.gif)


# 4\. 总结


`Streamlit` 与 `Matplotlib` 结合的关键点在于，`Streamlit` 提供便捷的应用构建框架，`Matplotlib` 专注强大的绘图功能，二者通过 `st.pyplot` 函数紧密相连。


在数据处理流程上，可先在 `Matplotlib` 中依据数据特性灵活创建各类图形对象，再借助 `Streamlit` 整合进应用。


它们结合的优势也非常明显，首先，在可视化效率方面，能快速将 `Matplotlib` 绘制的图形嵌入 `Streamlit` 应用，减少开发时间与代码量。


其次，在交互性上，`Streamlit` 的丰富交互组件可与 `Matplotlib` 图形联动，如通过按钮、滑块等控制图形展示内容或范围，让用户更方便的进行数据探索。


