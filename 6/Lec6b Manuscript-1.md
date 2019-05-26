# Lec6b Manuscript-1

## 引言

分离程序中时间表面上的顺序与机器中的实际计算顺序，也就是在需要的时候才生成其中的元素，称为按需计算（惰性计算）。



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



### 递归打印

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



### 复杂例子：获取整数域中的不能被7整除的数