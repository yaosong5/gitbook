[TOC]

[Numpy文档](https://docs.scipy.org/doc/numpy/reference/)

# 一、初始化

## 使用内建函数

**使用内建函数-创建一个数组**
a = np.zeros((2, 3))

a = np.ones((2, 5))

**创建全是8的数组**
a = np.full((2, 3), 8)
**创建对角的数组,相当于对角矩阵**
print(np.eye(3))
**创建一个随机的矩阵,值都是int**
a = np.random.randint(2, 7, (2, 3))

a = np.random.randn(6,3)

**创建一个随机数值，但是形状固定的矩阵数组**
a = np.random.random((2, 3))
a = np.empty((2, 3)))

## 直接实例化

**创建数组，设置数值类型**
a = np.array([1, 2, 4, 5], dtype=float)
print(a.dtype)
print(a.shape)

## 类型

[类型文档](https://docs.scipy.org/doc/numpy/reference/arrays.dtypes.html)

**转换数据类型**
a = np.array([1, 2, 4, 5], dtype=float).astype(int)
print(a.dtype)



1. 生成数组时可以指定数据类型，如果不指定numpy会**自动匹配**合适的类型
2. 使用astype**复制数组**并转换数据类型
3. 使用astype将float转换为int时**小数部分被舍弃**
4. 使用astype把字符串转换为数组，如果失败抛出异常。
5. astype可以使用其它数组的数据类型作为参数 arr1.astype(dtype=arr2.dtype)



# 二、取值赋值

a = np.array([[1, 2, 3, 4], [5, 6, 7, 8], [9, 10, 11, 12]])

## 1.切片取值

可以像list一样切片（多维数组可以从各个维度同时切片）

print(a.shape)
b = a[0:2, 2: 4].copy()
print(b)
print(a[:2, 1:])

也可以赋值

a[0,0]=2222

不建议你这样去赋值，但是你确实可以修改切片出来的对象，然后完成对原数组的赋值.

copy也相当于赋值

## 2.条件判定bool取值

很好用

print(a[a / 2 =0])

## 3.精准取值

print("精准取值")
print([a[0, 0], a[1, 2], a[2, 2]])



取值的样图：

![](https://ws2.sinaimg.cn/large/006tNc79gy1g03i8gk0ppj30za0sugrb.jpg)

# 三、运算

[详细运算文档](https://docs.scipy.org/doc/numpy/reference/routines.math.html) 比如比如矩阵的变形，转置和重排等等:

如果两个数组x,y维度一样，可以直接进行运算

- 逐元素求和，差，乘，除：x+y、np.add(x,y)，x-y、(np.subtract(x,y))，x*y 、(np.multiply(x,y))，x/y 、(np.divide(x, y))

- 逐元素求平方根：np.squrt(x)

- 求向量内积（也是矩阵的乘法）：np.dot(v,w)

- 转置：x.T

- 求和：np.sum(x)  np.sum(x,axis=0) 求列和  np.sum(x,axis=1) 求横排和也就是各个单元的和

- 求平均：np.mean(x) 同上

- 比如求和，求平均，求cumulative sum，sumulative product用numpy都可以做

  x.cumsum(axis=0)

  x.cumprod(axis=0)

- 排序：x.sort()

  - 找出排序后位置在5%的数字： 

    ```
    large_arr = np.random.randn(1000)
    large_arr.sort()
    print(large_arr[int(0.05 * len(large_arr))])
    ```

    ​



# 四、Broadcasting

如果要用小的矩阵去和大的矩阵做一些操作，但是希望小矩阵能循环和大矩阵的那些块做一样的操作，那急需要Broadcasting啦

我们要做一件事情，给x的每一行都逐元素加上一个向量，然后生成y

x = np.array([[1,2,3], [4,5,6], [7,8,9], [10, 11, 12]])

v = np.array([1, 0, 1])

x + np.reshape(w, (2,1))

broadcasting当然可以逐元素运算了

先把v变形成3x1的数组/矩阵，然后就可以broadcasting加在w上了

总结一下broadcasting，可以看看下面的图：

![](https://ws3.sinaimg.cn/large/006tNc79gy1g03i8x2hifj30sg0kqju6.jpg)



# 五、逻辑运算where

x_arr = np.array([1.1, 1.2, 1.3, 1.4, 1.5])

y_arr = np.array([2.1, 2.2, 2.3, 2.4, 2.5])

cond = np.array([True, False, True, True, False])

np.where(cond, x_arr, y_arr)

得到的数据：

[1.1 2.2 1.3 1.4 2.5]

如果满足条件，就第一个，不然就第二个，可以是取值，也可以是直接赋值。

np.where(arr > 0, 1,-1)





# 六、高级ndarray处理

- 使用reshape来改变tensor的形状，numpy可以很容易地把一维数组转成二维数组，三维数组。

arr.arange(8) 可以转换成arr.reshape(2,4) 也可以转换成arr.reshape(2,2,2)

- 如果我们在某一个维度上写上-1，numpy会帮我们自动推导出正确的维度还可以从其他的ndarray中获取shape信息然后reshape
- 高维数组可以用ravel来拉平：arr.ravel()
- 拼接二维数组：

np.concatenate([arr1, arr2], axis=0)

- 所谓堆叠，参考叠盘子。。。连接的另一种表述 垂直stack与水平stack

np.vstack((arr1, arr2)) # vertical

np.hstack((arr1, arr2)) # horizontal

- 堆叠辅助

  arr = np.arange(6)

  arr1 = arr.reshape((3, 2))

  arr2 = np.random.randn(3, 2)

  - r_用于按行堆叠
  - c_用于按列堆叠

```
np.r_[arr1, arr2]
np.c_[np.r_[arr1, arr2], arr]
```



- 拆分数组

  first, second, third = np.split(arr, [1,3], axis=0)

  - axis = 0：表示前面是行数

- 重复repeat （[详细介绍](https://docs.scipy.org/doc/numpy/reference/generated/numpy.tile.html)）

  - 按元素重复

  - 指定axis来重复

    arr.repeat(2, axis=1)



# 文件输入输出

- 读取csv文件：arr = np.loadtxt('array_ex.txt', delimiter=',')
- 保存数组文件：np.save('some_array', arr)

# 写一个softmax

[softmax解析](http://cs231n.github.io/linear-classify/#softmax)







