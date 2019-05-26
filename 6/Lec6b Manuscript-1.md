# Lec6b Manuscript-1

## 引言

分离程序中时间表面上的顺序与机器中的实际计算顺序，也就是在需要的时候才生成其中的元素，称为按需计算（惰性计算）。

**仅对 SICP 课程思想进行学习，并用 Python 语言实现课程中提及的知识，不探究在 Python 语言中的实用性**

## 按需计算程序

```python
# 上节的基本函数，在当节需要使用
def cons_stream(x, y):
    return [x, y]

def cons_empty_stream():
    return []

def get_head(s):
    if isinstance(s, Iterator):
        s = next(s)
    #     print(s)
    return s[0]

def get_tail(s):
    if isinstance(s, Iterator):
        s = next(s)

    return s[1]

def is_empty_stream(s):
    if isinstance(s, Iterator):
        s = next(s)

    if len(s) == 0:
        return True
    else:
        return False


def map_stream(proc, s):
    "对流使用 prco 递归处理"
    if is_empty_stream(s):
        return cons_empty_stream()
    else:
        return cons_stream(
            proc(get_head(s)),
            map_stream(proc, get_tail(s))
        )


def filter_stream(pred, s):
    "对流使用 pred 递归过滤处理"

    #     print(s)
    if isinstance(s, Iterator):
        s = next(s)
    if is_empty_stream(s):
        yield cons_empty_stream()
    else:
        if pred(get_head(s)):
            yield cons_stream(
                get_head(s),
                filter_stream(pred, get_tail(s))
            )
        else:
            yield filter_stream(pred, get_tail(s))


def accumulate(combiner, init_val, s):
    "组合累积"
    if is_empty_stream(s):
        return init_val
    else:
        result = combiner(
            get_head(s),
            accumulate(combiner, init_val, get_tail(s))
        )
        return result

```



### 简单例子

#### 获取第N个元素

````python
# 按需计算
from collections import Iterator
def nth_stream(n, s):
    if isinstance(s, Iterator):
        s = next(s)

    if n == 0:
        return get_head(s)
    else:
        return nth_stream(n-1, get_tail(s))

n = 2
s = [1,[2,[3,[]]]] # 模拟SICP CONS结构，类似链表的二元列表

result = nth_stream(n, s)
print(result)

# 控制台
>>> 3
````



#### 递归打印

``````python
def print_stream(s):
    if is_empty_stream(s):
        print("done")
    else:
        print(get_head(s))
        return print_stream(get_tail(s))
      
print_stream(s)

# 控制台
>>> 1
2
3
done
``````



### 复杂例子：获取整数域中的不能整除7的整除

``````python
# 由于 SICP 中使用的是 Lisp 语言，用 Python 替代需要做部分改造
# 如：延迟计算需要使用到 Python 中的 yield

# 构造获取整个整数序对的函数
def int_from(n):
    yield cons_stream(
        n,
        int_from(n+1))
    
    
def integers():
    return int_from(1)
  
result = nth_stream(20, integers()) # 打印第21个整数
print(result)

# 控制台
>>> 21
``````



``````python
# 过滤器：不能整除7的整数
def no_seven(n):
    if n % 7 == 0:
        return False
    else:
        return True
      
# 需要对基本函数作改造，以适应 yield
def get_head(s):
    if isinstance(s, Iterator):
        s = next(s)

    return s[0]

def get_tail(s):
    if isinstance(s, Iterator):
        s = next(s)

    return s[1]

def is_empty_stream(s):
    if isinstance(s, Iterator):
        s = next(s)

    if len(s) == 0:
        return True
    else:
        return False
      
def filter_stream(pred, s):
    "对流使用 pred 递归过滤处理"
    if isinstance(s, Iterator):
        s = next(s)

    if is_empty_stream(s):
        yield cons_empty_stream()
    else:
        if pred(get_head(s)):
            yield cons_stream(
                get_head(s),
                filter_stream(pred, get_tail(s))
            )
        else:
            yield filter_stream(pred, get_tail(s))
            
# ###################################################

# 第101个不能整除7的整数
result = nth_stream(100, filter_stream(no_seven, integers()))
print(result) 

# 控制台
>>> 117
``````



## "一下子"处理整个流

原来的过程是递归地，一次生成一个元素，再组合一起。

另一种观点是不逐一生成，而是一次处理整个流。

``````python
def add_stream(s1, s2):
    # if is_empty_stream(s1):
    #     return s2
    # if is_empty_stream(s2):
    #     return s1

    if isinstance(s1, Iterator):
        s1 = next(s1)
    if isinstance(s2, Iterator):
        s2 = next(s2)

    yield cons_stream(
        s1[0] + s2[0],
        add_stream(s1[1], s2[1])
    )

# 由1组成的无穷列表
def ones():
    yield cons_stream(1, ones())

# 获取整数序对的函数
def integers_2():
    yield cons_stream(1,
                   add_stream(integers_2(), ones()))
    
result = nth_stream(10, integers_2())
print(result)

# 控制台
>>> 11

# 可以看作是如下的电路图
"""
                               1 (延时元件)
ones --> 加法累加器 -----------  V  --------->integers_2(输出)
              ^                      |
  					  |                      |
  					  |----------------------| 
      					     (小型反馈回路)
    															
          
# 稍加改造可以计算流的积分
																			S0
	 |-------------------------------------------|
   | INTEGRATOR                        |       |
S --（* dt）---> 加法累加器 -----------  V  ---------> ∫ sdt(输出)
   |           		^                      |		 | 
   |					 	 	|                      | 		 |
   | 					 	 	|----------------------|     |
   |   					     (小型反馈回路)          	 	 |
   |-------------------------------------------|
"""
``````



### 斐波那契

``````python
# 隐藏计算非常玄乎
def fibs():
    yield cons_stream(
            0,
            cons_stream(1,
                add_stream(
                    fibs(),
                    get_tail(fibs()))))

result = nth_stream(10, fibs())
print(result)


# 控制台
>>> 55
``````

| Function       |      |      |      |      |      |
| -------------- | ---- | ---- | ---- | ---- | ---- |
| fibs           | 0    | 1    | 1    | 2    | ...  |
| get_tail(fibs) | 1    | 1    | 2    | 3    | ...  |
| Result         | 1    | 2    | 3    | 5    | ...  |

