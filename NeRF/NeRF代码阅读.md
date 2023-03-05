# NeRF 代码阅读

借助 chatgptgpt 对 pytorch 实现的 NeRF 做简要说明

## batchify 函数

```python
def batchify(fn, chunk):
    """Constructs a version of 'fn' that applies to smaller batches.
    """
    if chunk is None:
        return fn
    def ret(inputs):
        return torch.cat([fn(inputs[i:i+chunk]) for i in range(0, inputs.shape[0], chunk)], 0)
    return ret
```



## 知识点

### 1. c2w 相机坐标到世界坐标的变换

### 2. ndc 标准化坐标

