[toc]

# 引入 numpy 包

```python
import numpy as np
```





# 创建 `ndarray` 数组

**通过数组创建：**

```python
np.array([[1,2,3], [4,5,6]], dtype=np.float32)
```

**创建全 1，全 0 矩阵：**

```python
np.ones(shape=(3, 3)) # 全1

np.zeros(shape=(3, 3)) # 全0
```

**按照标准正态分布随机化：**

```python
np.random.randn(3, 3) # 3*3的标准正态分布矩阵
```

**创建等差 `ndarray` 数组：**

```python
np.arange(start, stop, step)
```





# 常用 `ndarray` 属性

- **`shape` 属性**：矩阵形状
- **`dtype` 属性**：数据类型





# 访问元素

## 使用索引访问元素

**索引：**

```python
arr_2d = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])

arr_2d[0, 1]  # 输出: 2 # 是一个[], 而不是[][]
```

**布尔索引：**

```python
arr = np.array([1, 2, 3, 4, 5])

arr[arr > 3]  # 输出: [4 5]
```

## 使用切片访问元素

```python
arr_2d = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])

arr_2d[:, 1]  # 输出: [2 5 8]
```





# `ndarray` 数组的常用操作

## 形状操作

```python
array = np.array([1, 2, 3, 4, 5, 6])

reshaped_array = array.reshape(2, 3) # 2*3的矩阵 # 输出: [[1,2,3], [4,5,6]]
```

## 多维数组转一维数组

```python
array_2d = np.array([[1, 2, 3], [4, 5, 6]])

flattened_array = array_2d.flatten() # 1维数组 # 输出: [1, 2, 3, 4, 5, 6]
```

## 转置

```python
matrix = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])

transposed_matrix = matrix.T # 输出: [[1,4,7], [2,5,8], [3,6,9]]
```

## 矩阵拆分

- **`axis` 参数**，表示按照哪个维度进行拆分
- **`indices_or_sections` 参数**，表示在指定维度上， 对于哪些 index 需要执行拆分操作

```python
matrix = np.array([[1, 2, 3, 4],
                   [5, 6, 7, 8],
                   [9, 10, 11, 12],
                   [13, 14, 15, 16]])

# 等价于np.hsplit(matrix, [2])
np.split(matrix, indices_or_sections=[2], axis=1) # 由于axis=1表示行方向, 就将行拆分为[:2]和[2:]两个部分
# 输出: [[1,2], [5,6], [9,10], [13,14]] 和 [[3,4], [7,8], [11,12], [15,16]]

# 等价于np.vsplit(matrix, [2])
np.split(matrix, indices_or_sections=[2], axis=0)
```

## 矩阵组合

- **`axis` 参数**，表示按照哪个维度进行拼接

```python
a = np.array([[1, 2, 3], [4, 5, 6]])
b = np.array([[7, 8, 9], [10, 11, 12]])

# 等价于np.vstack((a, b))
c = np.concatenate((a, b), axis=0) # 在列方向上进行拼接
# 输出: [[1, 2, 3], [4, 5, 6], [7, 8, 9], [10, 11, 12]]

# 等价于np.hstack((a, b))
d = np.concatenate((a, b), axis=1) # 在行方向上进行拼接
```

## 打乱数组

```python
arr = np.array([1, 2, 3, 4, 5])

np.random.shuffle(arr)
```





# 广播

**步骤 1**：如果两个数组的维度数上不相同，那么<u>*小维度数组会在最左边补 1*</u>

**步骤 2**：如果两个对应的维度数不相同，且<u>*其中一个维度数为 1*</u>，那么<u>*将 1 提升到另一个更高的维度数*</u>

**步骤 3**：如果两个对应的维度数不相同，且两个维度数<u>*都不为 1*</u>，那么<u>*报错*</u>





# I/O

## 从 `.csv` 文件中读取 `ndarray` 数组

```python
np.loadtxt('temp.csv', dtype=np.float32, delimiter=',')
```

