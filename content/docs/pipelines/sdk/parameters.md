+++
title = "流水线参数"
description = "在流水线组件中传递数据"
weight = 70
+++

[`kfp.dsl.PipelineParam` 
类](https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.dsl.html#kfp.dsl.PipelineParam)
表示一个未来会传递给流水线或是由任务产生的数据的引用。

你的流水线函数应该包含参数，以便之后可以在 Kubeflow Pipelines UI 中进行配置。

当你的流水线函数被调用时，每个函数参数都会是一个 `PipelineParam` 对象。
你可以将这些对象作为参数传递给组件来进行实例化并创建任务。`PipelineParam` 还可以表示您在流水线任务之间传递的中间值。
每个任务都有输出，你可以通过 `task.outputs[<output_name>]` 字典获取它们的引用。任务输出引用可以再次作为参数传递给其他组件。

在大多数情况下，你不需要手动构造 `PipelineParam` 对象。

以下代码示例显示了如何定义带参数的流水线：

```python
@kfp.dsl.pipeline(
  name='My pipeline',
  description='My machine learning pipeline'
)
def my_pipeline(
    my_num: int = 1000, 
    my_name: str = 'some text', 
    my_url: str = 'http://example.com'
):
  ...
```

在 [构建组件](/docs/pipelines/sdk/build-component/#create-a-python-class-for-your-component) 指南中查看更多信息。
