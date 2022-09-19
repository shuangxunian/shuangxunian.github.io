---
title: topk的五种求法
excerpt: 排序，局部排序，堆，基于快排，魔法
categories:
- 算法
tags:
- 面试
---

## 前言
昨天lc的每日一题是一道topk，用的最小堆，总觉得不止一个解决方法，昨天问问队友查查资料发现果然不是，这里做一下记录。

这里用lc一道典型题做例子：输入整数数组 arr ，找出其中最小的 k 个数。例如，输入4、5、1、6、2、7、3、8这8个数字，则最小的4个数字是1、2、3、4。[题目点此](https://leetcode-cn.com/problems/zui-xiao-de-kge-shu-lcof/)

## 排序
没啥好说的，直接上代码：

```javascript
var getLeastNumbers = function(arr, k) {
    return arr.sort((a, b) => a - b).slice(0, k);
};
```

## 局部排序
我们只要前k个，上个方法我们是把所有的全排序输出了，因此又能引出这个思路，只冒泡前k个，这里只留思路不留代码

## 堆
思路是只找到TopK，不排序TopK。先用前k个元素生成一个小顶堆，这个小顶堆用于存储当前的k个元素。接着，从第k+1个元素开始扫描，和堆顶(堆中最小的元素)比较，如果被扫描的元素大于堆顶，则替换堆顶的元素，并调整堆，以保证堆内的k个元素，总是当前最小的k个元素。直到扫描完所有n-k个元素，最终堆中的k个元素，就是用堆求的TopK，说白了就是把冒泡的TopK排序优化为了TopK不排序，节省了计算资源。

```javascript
var getLeastNumbers = function(arr, k) {
    let len = arr.length
    if (!len || !k) return []
    let heap = new Heap()
        // 建立最小堆，O(N) 复杂度
    heap.init(arr)
    let res = []
    while (k) {
        // 依次从堆顶弹出最小元素，O(logN) 复杂度
        res.push(heap.delete())
        k--
    }
    return res
}
function Heap() {
    this.heap = [-Infinity]
}
Heap.prototype.init = function(arr) {
    this.heap = [-Infinity]
    this.heap.push(...arr)
    let size = arr.length
        // 从最后一个元素的父节点开始实现最小堆，类似删除操作中将最后一个元素放在堆顶进行调整。
    for (let pos = parseInt(size / 2); pos > 0; pos--) {
        let tmp = this.heap[pos]
        let parent, child
        for (parent = pos; parent * 2 <= size; parent = child) {
            child = parent * 2
            if (child + 1 <= size && this.heap[child + 1] < this.heap[child]) child++
                if (tmp < this.heap[child]) break
                else this.heap[parent] = this.heap[child]
        }
        this.heap[parent] = tmp
    }
}
Heap.prototype.delete = function() {
    let size = this.heap.length - 1
    let res = this.heap[1]
        // 拿到最后一个元素
    let tmp = this.heap[size]
    this.heap.length--
        size--
        // 将最后一个元素放在堆顶，并调整最小堆
        let parent, child
    for (parent = 1; parent * 2 <= size; parent = child) {
        child = parent * 2
        if (child + 1 <= size && this.heap[child + 1] < this.heap[child]) child++
            if (tmp < this.heap[child]) break
            else this.heap[parent] = this.heap[child]
    }
    this.heap[parent] = tmp
    return res
}
```

## 基于快排
根据快排的思路，不断分治，如果我们不分治到最后求排序结果，只是划分到当当前的k大于等于当前数组的长度时，这个数组一定是满足要求的数组，直接将此数组返回即可。

```javascript
function partition(arr, start, end) {
    const k = arr[start];
    let left = start + 1,
        right = end;
    while (1) {
        while (left <= end && arr[left] <= k) ++left;
        while (right >= start + 1 && arr[right] >= k) --right;

        if (left >= right) {
            break;
        }

        [arr[left], arr[right]] = [arr[right], arr[left]];
        ++left;
        --right;
    }
    [arr[right], arr[start]] = [arr[start], arr[right]];
    return right;
}

var getLeastNumbers = function(arr, k) {
    const length = arr.length;
    if (k >= length) return arr;
    let left = 0,
        right = length - 1;
    let index = partition(arr, left, right);
    while (index !== k) {
        if (index < k) {
            left = index + 1;
            index = partition(arr, left, right);
        } else if (index > k) {
            right = index - 1;
            index = partition(arr, left, right);
        }
    }

    return arr.slice(0, k);
};
```

## 魔法
这个是别人发给我的，找不到具体链接，如果读者有链接可以联系我~

```c++
int findKthLargest(vector<int>& nums, int k)
{
	auto minmax = minmax_element(nums.begin(), nums.end());
	int left = *minmax.first;
	int right = *minmax.second;
	int mid;
	int countGreat;
	while (left < right)
	{
		mid = left + (right - left) / 2;
		countGreat = count_if(nums.begin(), nums.end(), [mid](int n) { return n> mid;});
		if (countGreat >= k)
		{
			left = mid + 1;
		}
		else
		{
			right = mid;
		}
	}
	return left;
}
```

## 参考
[最小的k个数](https://leetcode-cn.com/problems/zui-xiao-de-kge-shu-lcof/solution/zui-xiao-de-kge-shu-by-leetcode-solution/)
[拜托，面试别再问我TopK了](https://zhuanlan.51cto.com/art/201809/584259.htm)