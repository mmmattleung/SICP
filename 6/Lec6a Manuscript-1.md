# Lec6a Manuscript-1

## 引言

把事物运动看成是一个整体，不是离散的状态——流处理

## 函数式编程

### 二叉树遍历程序

计算二叉树中的奇数叶子结点的平方和

>PS：关注程序结构而非具体实现

```python
# 函数式编程
def sum_odd_square_tree(tree):
    def is_only_leafs(tree):
        '是否只有一个结点'
        pass

    def is_odd(leaf):
        '是否是奇数'
        pass

    def get_left_branch(tree):
        '获取左边分支'
        pass

    def get_right_branch(tree):
        '获取右边分支'
        pass

    if is_only_leafs(tree):
        if is_odd(tree):
            return tree * tree
        else:
            return 0
    else:
        return sum_odd_square_tree(get_left_branch(tree)) + \
            sum_odd_square_tree(get_right_branch(tree))
```

&nbsp;

###  斐波那契奇数列表

枚举1～N，计算其斐波那契对应的值，加入到列表中

```python
# 函数式编程
def fib(n):
    if n == 0:
        return 0
    elif n == 1 or n == 2:
        return 1
    else:
        return fib(n-1) + fib(n-2)
      
result_list = []
def odd_fibs(n):
    def _next(k):
        if k > n:
            return []
        else:
            tmp = fib(k)
            print(tmp)
            if tmp % 2 != 0:
                l.append(tmp)
            return _next(k+1)
    return _next(1)
```

&nbsp;

## 流编程

### 构造流

> SICP 中采用 LISP 语言，此处用 Python 语言代替
>
> "流"：假定是一个二元列表，用以模仿 Lisp 的结构，是一种类似链表的结构.[x, [y, [z, []]]]

``````python
# 构造函数
def cons_stream(x, y):
    return [x, y]

def cons_empty_stream():
    return []
``````

&nbsp;

``````python
# 选择函数
def get_head(s):
    return s[0]

def get_tail(s):
    return s[1]

def is_empty_stream(s):
    if len(s) == 0:
        return True
    else:
        return False
``````

### 操作流

``````python
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
    if is_empty_stream(s):
        return cons_empty_stream()
    else:
        if pred(get_head(s)):
            return cons_stream(
                get_head(s),
                filter_stream(pred, get_tail(s))
            )
        else:
            return filter_stream(pred, get_tail(s))

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
``````

### 二叉树遍历程序改造

把流式编程看成是一个数据结构通过多个盒子的处理，最后得到目标数据的一个过程。

begin ->

box-1：枚举树结点上的数字 -> 

box-2：过滤奇数部分 ->

box-3：奇数部分取平方 ->

box-4：求和 ->

end .

````python
# box-1
def enumerate_tree(tree):
    def is_only_leafs(tree):
        pass

    def get_left_branch(tree):
        pass

    def get_right_branch(tree):
        pass

    def append_stream(s1, s2):
        if is_empty_stream(s1):
            return s2
        else:
            cons_stream(
                get_head(s1),
                append_stream(get_tail(s1), s2)
            )

    if is_only_leafs(tree):
        return cons_stream(tree, [])
    else:
        return append_stream(
            enumerate_tree(get_left_branch(tree)),
            enumerate_tree(get_right_branch(tree))
        )
````

&nbsp;

``````python
# box-2
def is_odd(c):
    if c % 2 != 0:
        return True
    else:
        return False
``````

&nbsp;

``````python
# box-3
def square_p(s):
    return s*s
``````

&nbsp;

``````python
# box-4
def sum_p(s1, s2):
    return s1+s2
``````



``````python
# 组织程序
def sum_odd_square_tree_2(tree):
    return accumulate(
            sum_p, # box-4
            0,
            map_stream(
                square_p, # box-3
                filter_stream(is_odd, # box-2
                              enumerate_tree(tree)) # box-1
            )
        )
``````



### 斐波那契奇数列表改造

``````python
# ############################################
# 流-数据抽象
# 构造函数
def cons_stream(x, y):
    return [x, y]


def cons_empty_stream():
    return []


# 选择函数
def get_head(s):
    return s[0]


def get_tail(s):
    return s[1]


def is_empty_stream(s):
    if len(s) == 0:
        return True
    else:
        return False


# 对流的头部进行proc处理
# 递归尾部并且进行proc处理
# 合成新的流
def map_stream(proc, s):
    if is_empty_stream(s):
        return cons_empty_stream()
    else:
        return cons_stream(
            proc(get_head(s)),
            map_stream(proc, get_tail(s))
        )

# 对头进行过滤判断
# 递归尾部，进行过滤判断
# 组成新的流
def filter_stream(pred, s):
    if is_empty_stream(s):
        return cons_empty_stream()
    else:
        if pred(get_head(s)):
            return cons_stream(
                get_head(s),
                filter_stream(pred, get_tail(s))
            )
        else:
            return filter_stream(pred, get_tail(s))


# 累积函数
def accumulate(combiner, init_val, s):
    if is_empty_stream(s):
        return init_val
    else:
        a1 = combiner(
            get_head(s),
            accumulate(combiner, init_val, get_tail(s))
        )
        return a1
# ############################################

def fib(n):
    if n == 0:
        return 0
    elif n == 1 or n == 2:
        return 1
    else:
        return fib(n-1) + fib(n-2)

def enmu_interval(low, high):
    if low > high:
        return cons_empty_stream()
    else:
        return cons_stream(
            low,
            enmu_interval(low+1, high)
        )

def is_odd(c):
    if c % 2 != 0:
        return True
    else:
        return False

def t():
    # for i in range(1, 10):
    #     print(i, fib(i))

    # assert enmu_interval(1, 2) == [1, [2, []]]
    # assert enmu_interval(2, 6) == [2, [3, [4, [5, [6,[]]]]]]

    # assert map_stream(fib, [1, [2, []]]) == [1, [1, []]]
    # assert map_stream(fib, [2, [3, [4, [5, [6,[]]]]]]) == [1, [2, [3, [5, [8, []]]]]]

    # assert filter_stream(is_odd, [1, [1, []]]) == [1, [1, []]]
    # assert filter_stream(is_odd, [2, [3, [4, [5, [6, []]]]]]) == [3, [5, []]]

    print(
        accumulate(             # 累积
            cons_stream,        
            [], 
            filter_stream(
                is_odd,          # 过滤奇数
                map_stream(fib,  # 取斐波那契值
                           enmu_interval(2, 6)) # 枚举
            )
        )
    )

    print("Test Finished!")


t()
``````

