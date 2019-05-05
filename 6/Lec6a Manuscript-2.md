# Lec6a Manuscript-2

## 引言

改变建模观念— 不把事物看成是按时间演进，不再具有不同阶段和状态，而是对全局建模

## 复杂的例子

目标：

1. 枚举给定数字，构造二元列表
2. 求出每个内嵌列表的和(I+J)，并判断其和是否是质数
3. 如果是加入到内嵌二元列表中

``````python
N = 6

I     J     I+J
1     2      3
2     3      4
3     4      5
4     5      9
5     6      11

1. [1, 2, 3, 4, 5, 6]
2. [[1,2], [2, 3], [3, 4].....]
3. [[1, 2, 3], [2, 3, 5] .....]
``````



``````python
def enumera_helper(n):
    "枚举1～N列表"
    li = [i for i in range(1, n+1)]
    return li


def map_helper(data_list):
    "构造列表"
    return list(map(lambda x: [x, x+1], data_list[:-1]))


def flag_map(n):
    """
    :input: n
    :output: [[1, 2], [2, 3], [3, 4], [4, 6]...[n-1, n]]
    """
    return map_helper(
        enumera_helper(n)
    )


def is_prime(i):
    "判断质数"
    num = i[0]+i[1]
    if num > 1:
        for i in range(2, num):
            if (num % i) == 0:
                return False
        else:
            return True
    else:
        return False


def my_filter(pred, s):
    "过滤函数"
    if len(s) == 0:
        return cons_empty_stream()
    else:
        result = []
        for i in s:
            if pred(i):
                result.append(i)
        return result


def prime_sum_paris(n):
    "构造质数及其成分的列表"
    return list(            # 累加列表
        map(                # 映射目标结构
            lambda x:[x[0], x[1], x[0]+x[1]],
            my_filter(      # 过滤质数
                is_prime,
                flag_map(n) # 枚举
            )
        )
    )


if __name__ == '__main__':
    try:
        assert enumera_helper(10) == [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
        assert enumera_helper(3) == [1, 2, 3]

        assert map_helper([1, 2, 3]) == [[1, 2], [2, 3]]
        assert map_helper([2, 3, 4]) == [[2, 3], [3, 4]]
        assert map_helper([1, 2, 3, 4]) == [[1, 2], [2, 3], [3, 4]]

        assert flag_map(5) == [[1, 2], [2, 3], [3, 4], [4, 5]]
        assert flag_map(4) == [[1, 2], [2, 3], [3, 4]]
        assert flag_map(3) == [[1, 2], [2, 3]]

        assert my_filter(is_prime, flag_map(5)) == [[1, 2], [2, 3], [3, 4]]

        assert prime_sum_paris(2) == [[1, 2, 3]]
        assert prime_sum_paris(3) == [[1, 2, 3], [2, 3, 5]]
        print("TEST FINISHED!")
    except AssertionError as e:
        traceback.print_exc()
``````



## 回溯

