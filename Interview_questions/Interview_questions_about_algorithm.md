# 面试题系列算法

## 1. 排序算法
排序是算法的基础。

### 1.1 什么是排序算法的稳定性，有什么意义
假定在待排序的记录序列中，存在多个具有相同的关键字的记录，若经过排序，这些记录的相对次序保持不变，即在原序列中，ri = rj，且 ri 在 rj 之前，当序列再次排序后，ri 仍在 rj 之前，则称这种排序算法是稳定的，否则不稳定。例如：淘宝搜索，已经按价格排序，现在又需要对结果进行销量排序，那么稳定的排序可以让销量相同的商品依旧保持着价格高低的排序展展示，只有销量不同才会重新排序。

稳定算法必须对算法实现进行分析，从而得到稳定的特性。不稳定的算法在某种条件下可以变得稳定，而稳定的算法在某种条件下也可以变为不稳定的算法。例如：

```c
void bubbleSort(Datatype a[], int n) {
	int i, j, flag = 1;
	DataType temp;
	for (i = 1; i < n && flag == 1; i++) {
		flag = 0;
		for (j = 0; j < n - 1; j++) {
			if (a[j].key > a[j + 1].key) { // 如果将条件由 “>” 改为 “>=” 就不稳定了。
				flag = 1;
				temp = a[j];
				a[j] = a[j + 1];
				a[j + 1] = temp;
			}
		}
	}
}

```

### 1.2 常见排序算法稳定对比，时间复杂度

常见的不稳定排序算法：

- 堆排序
- 快速排序
- 希尔排序
- 直接选择排序

常见的稳定排序算法：

- 基数排序
- 冒泡排序
- 直接插入排序
- 折半插入排序
- 归并排序

### 1.3 常见排序算法的实现
下面罗列一些常见排序算法的实现
#### 1.3.1 冒泡排序

|最坏时间复杂度|平均时间复杂度| 最优时间复杂度|辅助空间复杂度|
|:----------:|:---------:|:-----------:|:----------:|
|O(n²)       |O(n)       |O(n²)        |O(1)        |

```swift
func bubbleSort(_ nums: inout [Int]) {
    for i in 0..<nums.count - 1 {
        for j in 0..<nums.count - 1 - i {
            if nums[j] > nums[j + 1] {
                nums.swapAt(j, j + 1)
            }
        }
    }
}
```

#### 1.3.2 插入排序

|最坏时间复杂度|平均时间复杂度| 最优时间复杂度|辅助空间复杂度|
|:----------:|:---------:|:-----------:|:----------:|
|O(n²)       |O(n)       |O(n²)        |O(1)        |

![插入排序](https://upload.wikimedia.org/wikipedia/commons/thumb/0/0f/Insertion-sort-example-300px.gif/220px-Insertion-sort-example-300px.gif)

```swift
func insertionSort(_ nums: inout [Int]) {
    guard nums.count >= 1 else { return }
    for i in 1..<nums.count {
        let n = nums[i]
        for j in (0..<i).reversed() {
            if nums[j] > n {
                nums.swapAt(j, j + 1)
            }
        }
    }
}
```

#### 1.3.3 选择排序

|最坏时间复杂度|平均时间复杂度| 最优时间复杂度|辅助空间复杂度|
|:----------:|:---------:|:-----------:|:----------:|
|O(n²)       |O(n²)      |O(n²)        |O(1)        |

![选择排序](https://upload.wikimedia.org/wikipedia/commons/9/94/Selection-Sort-Animation.gif)

```swift
func selectionSort(_ nums: inout [Int]) {
    for i in 0..<nums.count - 1 {
        var minIndex = i
        for j in i..<nums.count {
            if nums[minIndex] > nums[j] {
                minIndex = j
            }
        }
        nums.swapAt(i, minIndex)
    }
}
```

#### 1.3.4 快速排序
快速排序（英语：Quicksort），又称划分交换排序（partition-exchange sort），简称快排，一种排序算法，最早由东尼·霍尔提出

快速排序是二叉查找树（二叉搜索树）的一个空间最优化版本。不是循序地把数据项插入到一个明确的树中，而是由快速排序组织这些数据项到一个由递归调用所隐含的树中。这两个算法完全地产生相同的比较次数，但是顺序不同。对于排序算法的稳定性指标，原地分割版本的快速排序算法是不稳定的。其他变种是可以通过牺牲性能和空间来维护稳定性的。
快速排序的最直接竞争者是堆排序（Heapsort）。堆排序通常比快速排序稍微慢，但是最坏情况的运行时间总是 O(nlogn)。快速排序是经常比较快，除了introsort变化版本外，仍然有最坏情况性能的机会。如果事先知道堆排序将会是需要使用的，那么直接地使用堆排序比等待introsort再切换到它还要快。堆排序也拥有重要的特点，仅使用固定额外的空间（堆排序是原地排序），而即使是最佳的快速排序变化版本也需要 O(logn) 的空间。然而，堆排序需要有效率的随机存取才能变成可行。

快速排序也与归并排序（Mergesort）竞争，这是另外一种递归排序算法，但有坏情况 O(nlogn)运行时间的优势。不像快速排序或堆排序，归并排序是一个稳定排序，且可以轻易地被采用在链表（linked list）和存储在慢速访问媒体上像是磁盘存储或网络连接存储的非常巨大数列。尽管快速排序可以被重新改写使用在链串列上，但是它通常会因为无法随机存取而导致差的基准选择。归并排序的主要缺点，是在最佳情况下需要 Ω(n)额外的空间

|最坏时间复杂度|平均时间复杂度| 最优时间复杂度|辅助空间复杂度|
|:----------:|:---------:|:-----------:|:----------:|
|O(n²)       |O(nlogn)   |O(nlogn)     |依实现方式而定|

![快速排序](https://upload.wikimedia.org/wikipedia/commons/6/6a/Sorting_quicksort_anim.gif)

迭代方式：

```c
typedef struct _Range {
    int start, end;
} Range;

Range new_Range(int s, int e) {
    Range r;
    r.start = s;
    r.end = e;
    return r;
}

void swap(int *x, int *y) {
    int t = *x;
    *x = *y;
    *y = t;
}

void quick_sort(int arr[], const int len) {
    if (len <= 0)
        return; // 避免len等於負值時引發段錯誤（Segment Fault）
    // r[]模擬列表,p為數量,r[p++]為push,r[--p]為pop且取得元素
    Range r[len];
    int p = 0;
    r[p++] = new_Range(0, len - 1);
    while (p) {
        Range range = r[--p];
        if (range.start >= range.end)
            continue;
        int mid = arr[(range.start + range.end) / 2]; // 選取中間點為基準點
        int left = range.start, right = range.end;
        do {
            while (arr[left] < mid) ++left;   // 檢測基準點左側是否符合要求
            while (arr[right] > mid) --right; //檢測基準點右側是否符合要求
            if (left <= right) {
                swap(&arr[left], &arr[right]);
                left++;
                right--;               // 移動指針以繼續
            }
        } while (left <= right);
        if (range.start < right) r[p++] = new_Range(range.start, right);
        if (range.end > left) r[p++] = new_Range(left, range.end);
    }
}
```

递归方式：

```c
typedef struct _Range {
    int start, end;
} Range;

Range new_Range(int s, int e) {
    Range r;
    r.start = s;
    r.end = e;
    return r;
}

void swap(int *x, int *y) {
    int t = *x;
    *x = *y;
    *y = t;
}

void quick_sort(int arr[], const int len) {
    if (len <= 0)
        return; // 避免len等於負值時引發段錯誤（Segment Fault）
    // r[]模擬列表,p為數量,r[p++]為push,r[--p]為pop且取得元素
    Range r[len];
    int p = 0;
    r[p++] = new_Range(0, len - 1);
    while (p) {
        Range range = r[--p];
        if (range.start >= range.end)
            continue;
        int mid = arr[(range.start + range.end) / 2]; // 選取中間點為基準點
        int left = range.start, right = range.end;
        do {
            while (arr[left] < mid) ++left;   // 檢測基準點左側是否符合要求
            while (arr[right] > mid) --right; //檢測基準點右側是否符合要求
            if (left <= right) {
                swap(&arr[left], &arr[right]);
                left++;
                right--;               // 移動指針以繼續
            }
        } while (left <= right);
        if (range.start < right) r[p++] = new_Range(range.start, right);
        if (range.end > left) r[p++] = new_Range(left, range.end);
    }
}
```

快排的时间复杂度，为什么是 nlogn，最坏的情况是什么，最坏时间复杂度如何？

#### 1.3.5 归并排序
归并排序（英语：Merge sort，或mergesort），是创建在归并操作上的一种有效的排序算法，效率为 O(nlogn)。1945年由约翰·冯·诺伊曼首次提出。该算法是采用分治法（Divide and Conquer）的一个非常典型的应用，且各层分治递归可以同时进行

|最坏时间复杂度|平均时间复杂度| 最优时间复杂度|辅助空间复杂度|
|:----------:|:---------:|:-----------:|:----------:|
|O(nlogn)    |O(nlogn)   |O(nlogn)     |O(n)        |

![归并排序](https://upload.wikimedia.org/wikipedia/commons/thumb/c/cc/Merge-sort-example-300px.gif/220px-Merge-sort-example-300px.gif)


##### 迭代法（Bottom-up）

1. 将序列每相邻两个数字进行归并操作，形成 ceil(n/2)个序列，排序后每个序列包含两/一个元素
2. 若此时序列数不是1个则将上述序列再次归并，形成 ceil(n/4)个序列，每个序列包含四/三个元素
3. 重复步骤2，直到所有元素排序完毕，即序列数为1

迭代实现

```c
int min(int x, int y) {
    return x < y ? x : y;
}
void merge_sort(int arr[], int len) {
    int *a = arr;
    int *b = (int *) malloc(len * sizeof(int));
    int seg, start;
    for (seg = 1; seg < len; seg += seg) {
        for (start = 0; start < len; start += seg * 2) {
            int low = start, mid = min(start + seg, len), high = min(start + seg * 2, len);
            int k = low;
            int start1 = low, end1 = mid;
            int start2 = mid, end2 = high;
            while (start1 < end1 && start2 < end2)
                b[k++] = a[start1] < a[start2] ? a[start1++] : a[start2++];
            while (start1 < end1)
                b[k++] = a[start1++];
            while (start2 < end2)
                b[k++] = a[start2++];
        }
        int *temp = a;
        a = b;
        b = temp;
    }
    if (a != arr) {
        int i;
        for (i = 0; i < len; i++)
            b[i] = a[i];
        b = a;
    }
    free(b);
}
```

##### 递归法

1. 申请空间，使其大小为两个已经排序序列之和，该空间用来存放合并后的序列
2. 设定两个指针，最初位置分别为两个已经排序序列的起始位置
3. 比较两个指针所指向的元素，选择相对小的元素放入到合并空间，并移动指针到下一位置
4. 重复步骤3直到某一指针到达序列尾
5. 将另一序列剩下的所有元素直接复制到合并序列尾

递归实现：

```c
void merge_sort_recursive(int arr[], int reg[], int start, int end) {
    if (start >= end)
        return;
    int len = end - start, mid = (len >> 1) + start;
    int start1 = start, end1 = mid;
    int start2 = mid + 1, end2 = end;
    merge_sort_recursive(arr, reg, start1, end1);
    merge_sort_recursive(arr, reg, start2, end2);
    int k = start;
    while (start1 <= end1 && start2 <= end2)
        reg[k++] = arr[start1] < arr[start2] ? arr[start1++] : arr[start2++];
    while (start1 <= end1)
        reg[k++] = arr[start1++];
    while (start2 <= end2)
        reg[k++] = arr[start2++];
    for (k = start; k <= end; k++)
        arr[k] = reg[k];
}

void merge_sort(int arr[], const int len) {
    int reg[len];
    merge_sort_recursive(arr, reg, 0, len - 1);
}
```

#### 1.3.6 堆排序
堆排序（英语：Heapsort）是指利用堆这种数据结构所设计的一种排序算法。堆是一个近似完全二叉树的结构，并同时满足堆积的性质：即子节点的键值或索引总是小于（或者大于）它的父节点

|最坏时间复杂度|平均时间复杂度| 最优时间复杂度|辅助空间复杂度|
|:----------:|:---------:|:-----------:|:----------:|
|O(nlogn)    |O(nlogn)   |O(nlogn)     |O(1)        |

![堆排序](https://upload.wikimedia.org/wikipedia/commons/1/1b/Sorting_heapsort_anim.gif)

```c
#include <stdio.h>
#include <stdlib.h>

void swap(int *a, int *b) {
    int temp = *b;
    *b = *a;
    *a = temp;
}

void max_heapify(int arr[], int start, int end) {
    // 建立父節點指標和子節點指標
    int dad = start;
    int son = dad * 2 + 1;
    while (son <= end) { // 若子節點指標在範圍內才做比較
        if (son + 1 <= end && arr[son] < arr[son + 1]) // 先比較兩個子節點大小，選擇最大的
            son++;
        if (arr[dad] > arr[son]) //如果父節點大於子節點代表調整完畢，直接跳出函數
            return;
        else { // 否則交換父子內容再繼續子節點和孫節點比較
            swap(&arr[dad], &arr[son]);
            dad = son;
            son = dad * 2 + 1;
        }
    }
}

void heap_sort(int arr[], int len) {
    int i;
    // 初始化，i從最後一個父節點開始調整
    for (i = len / 2 - 1; i >= 0; i--)
        max_heapify(arr, i, len - 1);
    // 先將第一個元素和已排好元素前一位做交換，再重新調整，直到排序完畢
    for (i = len - 1; i > 0; i--) {
        swap(&arr[0], &arr[i]);
        max_heapify(arr, 0, i - 1);
    }
}

int main() {
    int arr[] = { 3, 5, 3, 0, 8, 6, 1, 5, 8, 6, 2, 4, 9, 4, 7, 0, 1, 8, 9, 7, 3, 1, 2, 5, 9, 7, 4, 0, 2, 6 };
    int len = (int) sizeof(arr) / sizeof(*arr);
    heap_sort(arr, len);
    int i;
    for (i = 0; i < len; i++)
        printf("%d ", arr[i]);
    printf("\n");
    return 0;
}
```

## 2. 数组
数组算法中常用到的概念：

- 中位数：有序序列最中间的那个数。如果序列的元素个数为偶数，那么此时中位数是最中间的两个元素的算数平均数


### 2.1 数组滑动窗口窗口的中位数？

## 3. 字符串

### 3.1 字符串旋转

### 3.2 字符串转浮点数

### 3.3 KMP 算法

## 4. 链表

### 4.1 链表反转

```swift
class ListNode {
    var val: Int
    var next: ListNode?
    
    init(_ val: Int = 0) {
        self.val = val
    }
}

// 三指针方法
class Solution1 {
    func reverseList(_ head: ListNode?) -> ListNode? {
        guard head?.next != nil else {
            return head
        }
        
        var previous = head
        var node = head?.next
        var next: ListNode?
        previous?.next = nil
        
        while node?.next != nil {
            next = node?.next
            node?.next = previous
            previous = node
            node = next
            next = nil
        }
        node?.next = previous
        return node
    }
}

// 递归的方式
class Solution2 {
    func reverseList(_ head: ListNode?) -> ListNode? {
        guard head?.next != nil else {
            return head
        }
        let h = reverseList(head?.next)
        head?.next?.next = head
        head?.next = nil
        
        return h
    }
}

```

### 4.2 找到链表倒数第 K 个结点

```swift
class ListNode {
    var val: Int
    var next: ListNode?
    
    init(_ val: Int = 0) {
        self.val = val
    }
}

class Solution {
    // 双指针
    func findKthFromEnd(_ head: ListNode?, _ k: Int) -> ListNode? {
        guard let node = head, node.next != nil else { return nil }
        
        var first = node, second = node
        var step = 0;
        
        while first.next != nil {
            first = first.next!
            step += 1
            if step > k {
                step = k
                second = second.next!
            }
        }
        return second
    }
}
```

### 4.3 链表环判断

```Java
class ListNode {
    int val;
    ListNode next;
    Listnode(int x) {
    	val = x;
    	next = null;
    }
}

// 快慢指针
public class Solution {
    public boolean hasCycle(ListNode head) {
    	if (head == null) {
    		return false;
    	}
    	ListNode fast = head;
    	ListNode slow = Head;
    	while ( fast != null ) {
    		if (fast.next == null) { return false }
    		fast = fast.next.next;
    		slow = slow.next;
    		if (fast == slow ) {
    			return true;
    		}
    	}
    	return false
}
```

### 4.4 链表相交

```swift
class ListNode {
    var val: Int
    var next: ListNode?
    
    init(_ val: Int = 0) {
        self.val = val
    }
}

class Solution {
    public func getIntersectionNode(_ headA: ListNode, _ headB: ListNode) -> ListNode? {
        var nodeA:ListNode? = headA
        var nodeB:ListNode? = headB
        /*
         该算法非常简洁，通过触底交换，巧妙地同步了两个长短不一的链表上的引用。
         假设 B 链较长，nodeA 将先抵达末尾，nodeA 指向 B 链，之后随着 nodeB 一起移动，
         当 NodeB 抵达末尾时，nodeA正好移动到与 A 等长（由尾至头）的位置，此时 NodeB 在
         指向 A 链，之后一起移动，引用相等时即为目标结点。
        */
        while nodeA !== nodeB {
            nodeA = nodeA == nil ? headB : nodeA?.next
            nodeB = nodeB == nil ? headA : nodeB?.next
        }
        return nodeA
    }
}
```

### 4.5 合并 K 个有序链表
先降维，然后利用归并法，先分后合。

```swift
class ListNode {
    var val: Int
    var next: ListNode?
    
    init(_ val: Int = 0) {
        self.val = val
    }
}

class Solution {
    func mergeKLists(_ lists: [ListNode?]) -> ListNode? {
        guard lists.count > 1 else { return lists.first as? ListNode }
        return partition(lists, left: 0, right: lists.count - 1)
    }
    
    // 归并合并2个有序链表，很简单，K 个就复杂了，怎么办呢？
    // 降维，把它拆分称2个的小问题
    func partition(_ lists: [ListNode?], left: Int, right: Int) -> ListNode? {
        guard left < right else { return lists[left] }
        let mid = (left + right) / 2
        // 递归拆分
        let l = partition(lists, left: left, right: mid)
        let r = partition(lists, left: mid + 1, right: right)
        // 递归合并
        return mergeTwoLists(l, r)
    }
    
    func mergeTwoLists(_ l1: ListNode?, _ l2: ListNode?) -> ListNode? {
        guard l1 != nil else { return l2 }
        guard l2 != nil else { return l1 }
        if l1!.val > l2!.val {
            let temp = l2
            temp?.next = mergeTwoLists(l1, l2?.next)
            return temp
        } else {
            let temp = l1
            temp?.next = mergeTwoLists(l2, l1?.next)
            return temp
        }
    }
}
```

## 5. 树

### 5.1 前、中、后序遍历的递归实现


### 5.2 前、中、后序遍历的迭代实现

### 5.3 Z 型打印二叉树

### 5.4 广度优先搜索

### 5.5 深度优先搜索


## 6. 动态规划

## 7. 其他
除去一些常规的算法类型，还有一些奇技淫巧，这里仅罗列一些常见。

### 7.1 不使用用 temp 交换 a、b 值

```swift
func swipTwoInt(_ a: inout Int, _ b: inout Int) {
    b = b ^ a // 取两数之间的不同，并赋值给 b
    a = a ^ b // 取 a 与不同的不同，即原 b 的值，
    b = a ^ b // 同理，再取原 b 与不同的不同，即原 a 的值
}
```

> 该问题关键在于对异或的理解，取不同的不同即是对方。

### T9算法如何实现，全拼算法

### 强连通量算法

### 最短路径算法

### 打印螺旋矩阵



### 银行家算法
银行家算法（Banker's Algorithm）是一个避免死锁（Deadlock）的著名算法，是由艾兹格·迪杰斯特拉在1965年为T.H.E系统设计的一种避免死锁产生的算法。它以银行借贷系统的分配策略为基础，判断并保证系统的安全运行

#### 小根堆的插入时间负责度

#### 二分查找的时间复杂度怎么求的？

#### 2个集合的交集

#### 找出数组中重复的数字

#### 百亿数据中查找相同的数字及出现次数

#### 在一个10G的数据中找到最大的100个数

#### 1000瓶药，10只狗，找出毒药。

#### atoi函数实现

#### n个整数的无序数组，找到每个元素后面比它大的第一个数，要求时间复杂度为O(N)

#### 百万数据找出前 K 个