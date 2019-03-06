[TOC]

[pandas官网](http://pandas.pydata.org/)



# 一、Series

- Series是一个一维的数据结构



## 赋值

- pandas会默认用0到n来作为Series的index，但是我们也可以自己指定index

```python
s = pd.Series([7, 'Beijing', 2.17, -1232, 'Happy birthday!'])
```

- 还可以用dictionary来构造一个Series，因为Series本来就是key value pairs。

```python
cities = {'Beijing': 55000, 'Shanghai': 60000, 'Shenzhen': 50000, 'Hangzhou':20000, 
'Guangzhou': 25000, 'Suzhou': None}
apts = pd.Series(cities)
```

前面定义的**index**就是用来选择数据的

apts[apts <= 50000] = 40000



## 取值

boolean indexing也可以用到pandas中来取值

apts[apts < 50000]

## 数学运算

- apts / 2
- apts * 2
- np.square(apts)
  - square也可以写成 **  ：apts ** 2

## 数据缺失

- apts.notnull()

- apts.isnull()： 

  - apts[apts.isnull()]  获取值为空的key，value
  - apts[apts.isnull() == False] 获取值不为空的key，value

  ​





# 二、DataFrame

[dataframe详细解析](http://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html)

一个Dataframe就是一张表格，Series表示的是一维数组，Dataframe则是一个二维数组，可以类比成一张excel的spreadsheet。也可以把Dataframe当做一组Series的集合。

## 创建一个DataFrame

dataframe可以由一个dictionary构造得到。

```python
data = {'city': ['Beijing', 'Shanghai', 'Guangzhou', 'Shenzhen', 'Hangzhou', 'Chongqing'],'year': [2016,2017,2016,2017,2016, 2016],'population': [2100, 2300, 1000, 700, 500, 500]}
pd.DataFrame(data)
```

columns的名字和顺序可以指定

```python
pd.DataFrame(data, columns = ['year', 'city', 'population'])
```

## 从DataFrame里选择数据

- frame2['city'] = frame2.city
- frame2.ix['three']： 这种是选取行index
- frame2.ix[2] ：这种方法默认用来选列而不是选行
- 还可以用Series来指定需要修改的index以及相对应的value，没有指定的默认用NaN.

```python
val = pd.Series([200, 300, 500], index=['two', 'three', 'five'])
```

- 指定index的顺序，以及使用切片初始化数据

```Python
frame3['Beijing'][1:3]
frame3['Shanghai'][:-1]
pdata = {'Beijing': frame3['Beijing'][:-1], 'Shanghai':frame3['Shanghai'][:-1]}
```

- 我们还可以指定index的名字和列的名字

```Python
frame3.index.name = 'year'
frame3.columns.name = 'city'
```



## DataFrame元素赋值

```Python
fram2["population"]["one"] = 2200
```



## 其他运算

- 一个DataFrame就和一个numpy 2d array一样，可以被转置： frame2.T



# 三、Index, reindex and hierarchical indexing

## index objects

- index的值是不能被更改的

- 针对index进行索引和切片

  obj = pd.Series(np.arange(4), index=['a','b','c','d'])

  默认的数字index依旧可以使用

- 如何对Series进行切片

- 对DataFrame进行Indexing与Series基本相同

  ```
  frame = pd.DataFrame(np.arange(9).reshape(3,3),
  index = ['a', 'c', 'd'],
  columns = ['Hangzhou', 'Shezhen', 'Nanjing'])
  ```

- [reindexs](http://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.reindex.html)

  把一个Series或者DataFrame按照新的index顺序进行重排

  ```
  obj = pd.Series([4.5, 7.2, -5.3, 3.2], index=['d', 'b', 'a', 'c'])
  obj.reindex(['a', 'b', 'c', 'd', 'e'])
  obj.reindex(['a', 'b', 'c', 'd', 'e'], fill_value = 0)
  ```

  ​

- 在reindex的同时，我们还可以重新指定columns

  ```Python
  frame.reindex(columns = ['Shenzhen', 'Hangzhou', 'Chongqing'])
  ```

  ​

- 可以用drop来删除Series和DataFrame中的index

  obj3.drop(2)

  obj3.drop([2, 4])


- unstack和stack可以帮助我们在hierarchical indexing和DataFrame之间进行切换。

  data.unstack()

  ​

# 四、Merge, Join, Concatenate, Groupby and Aggregate

## Merge(join)

pd.merge(df1, df4, on='cities')

- join on index

  df1.join(df4)

  df1.join(df4, how='outer')

  也可以用merge来写：pd.merge(df1, df4, left_index=True, right_index=True, how='outer')

## concatenate

在concatenate的时候可以指定keys，这样可以给每一个部分加上一个Key。

以下的例子就构造了一个hierarchical index。相当于再加一级index

```python
result2 = pd.concat(frames, keys=['x', 'y', 'z'])
```

![](https://ws4.sinaimg.cn/large/006tNc79gy1g04u1hrr0jj30gy07c74l.jpg)


- 用inner可以去掉NaN

  ```python
  result3 = pd.concat([result, df4], axis=1, join='inner')
  ```

  ​

- 用append来做concatenation

  df1.append(df4)

  可以通过append添加一个row

  ```python
  s2 = pd.Series([18000, 120000], index=['apts', 'cars'], name='Xiamen'
  df1.append(s2)
  ```





- Series和DataFrame还可以被一起concatenate，这时候Series会先被转成DataFrame然后做Join，因为Series本来就是一个只有一维的DataFrame对吧。




## Group By

groupby经常和aggregate一起使用



## describe

describe这个function可以为我们展示各种有用的统计信息



bikes.dropna()  删除空值

dropna会删除所有带NA的行

# 五、Read from csv

[读取csv文件](http://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.read_csv.html)

# 六、随堂案例：bikes routes counts