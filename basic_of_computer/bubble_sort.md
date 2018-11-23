# 冒泡排序

数组长度为n 

对数组遍历n次

每一次遍历将最大的元素移动到最后

## 示例

数组为[53, 23, 9, 17, 42]

### 第一次遍历过程：

1. 初始索引在第一个位置，index=0, 当前位置元素和下一个元素进行比较，即 53 和 23 比较，53 > 23。交换位置。 交换之后的数组为`[23，53，9，17，42]`

2. 索引向后移动，index=1。array[1] = 53，和下一个元素比较，即 53 和 9比较。53 > 9。交换位置。此时的数组为`[23, 9, 53, 17, 42]`

3. 索引向后移动，index=2。array[2] = 53。53 和 17比较，53 > 17。交换位置。此时数组 `[23, 9, 17, 53, 42]`

4. 索引继续向后移动，index = 3, array[3] = 53。53 和 42比较。53 > 42。交换位置。数组变成 `[23, 9, 17, 42, 53]`

5. 索引向后移动，index=4。索引值等于数组长度减一，即移动到了最后一个位置。这一次遍历完成。

第一次遍历之后，最大的元素53移动到了数组最后。

## CODE

```
def sort(array):

    for count in range(0,len(array)):

        # 遍历数组
        for index in range(0, len(array)-1):
            if array[index] > array[index+1]:
                temp = array[index+1]
                array[index+1] = array[index]
                array[index] = temp

        print(array)
        count = count  + 1
```

## 优化

1. 每次遍历过程逐次减少比较元素

因为每一轮遍历之后，最大的元素会移动到最后，所以下一次遍历最大的元素就不需要参加比较。

```
def sort(array):

    for count in range(0,len(array)):

        # 遍历数组
        for index in range(0, len(array)-1 - count):
            if array[index] > array[index+1]:
                temp = array[index+1]
                array[index+1] = array[index]
                array[index] = temp

        print(array)
        count = count  + 1
```

2. 一旦数组有序了，就停止继续遍历

```
def sort(array):
    for count in range(0,len(array)):

        array_sorted = True
        for index in range(0, len(array)-1):
            if array[index] > array[index+1]:
                temp = array[index+1]
                array[index+1] = array[index]
                array[index] = temp

                array_sorted = False

        if array_sorted:
            break

        print(array)
        count = count  + 1
```
