# 面试题系列之数据结构

## 1. 基础数据结构的实现

### 1.1 可变数组的实现

```c
#include <stdio.h>
#include <stdlib.h>
#include "MutableArray.h"


// 创建数组
Array array_create(int init_size) {
	Array a;
	a.array = (int *)malloc(sizeof(int) * init_size);
	a.size = init_size;
	return a;
}

// 释放数组的内存空间
void array_free(Array *a) {
	free(a->array);
	a->array = NULL;
	a->size = 0;
}

// 剩余可用空间
int array_size(Array *a) {
	return a->size;
}

// 访问数组中某个元素，可读可写
int * array_at(Array *a, int index) {
	if (index >= a->size) {
		array_inflate(a, (index / BLOCK_SIZE  + 1) * BLOCK_SIZE - a->size)
	}
	return &(a->array[index]);
}

// 数组扩容
void array_inflate(Array *a, int more_size) {
	int *p = (int *)malloc(sizeof(int) * (a->size + more_size));
	int i;
	// 将原空间的内容全部拷贝至新空间
	for ( i = 0; i < a->size; ++i)
	{
		p[i] = a->array[i];
	}
	
	// array_free(a);
	free(a->array);
	a->array = p;
	a->size += more_size;
}

// 向数组中写入
void array_set(Array *a, int index, int value) {
	a->array[index] = value;
}
```
引自 [用C语言简单实现一个可变数组](https://blog.csdn.net/melody_1016/article/details/81948809)
### 1.2 如何实现一个可变的哈希表/等同于如何实现一个 MutilableDictionary？

哈希表的难点在于如何将键与值的位置映射起来，以及如何解决哈希碰撞问题

```swift
public struct HashTable<Key: Hashable, Value> {
    private typealias Element = (key: Key, value: Value)
    private typealias Bucket = [Element]
    private var buckets: [Bucket]

    /// 哈希表中的键值对数量.
    private(set) public var count = 0

    /// 标识哈希表是否为空
    public var isEmpty: Bool { return count == 0 }

    /**
     创建一个指定容量的哈希表
     */
    public init(capacity: Int) {
        assert(capacity > 0)
        buckets = Array<Bucket>(repeatElement([], count: capacity))
    }

    /**
     下标方法：通过指定的键读/写相应的值
     */
    public subscript(key: Key) -> Value? {
        get {
            return value(forKey: key)
        }
        set {
            if let value = newValue {
                updateValue(value, forKey: key)
            } else {
                removeValue(forKey: key)
            }
        }
    }

    /**
     取值方法
     */
    public func value(forKey key: Key) -> Value? {
        let index = self.index(forKey: key)
        for element in buckets[index] {
            if element.key == key {
                return element.value
            }
        }
        return nil  // 不存在返空
    }

    /**
     根据指定的键更新或者添加一个值.
     */
    @discardableResult public mutating func updateValue(_ value: Value, forKey key: Key) -> Value? {
        let index = self.index(forKey: key)

        // 是否已经存在?
        for (i, element) in buckets[index].enumerated() {
            if element.key == key {
                let oldValue = element.value
                buckets[index][i].value = value
                return oldValue
            }
        }

        // 如果不存在，添加到链中.
        buckets[index].append((key: key, value: value))
        count += 1
        return nil
    }

    /**
     根据指定的键移除响应的值.
     */
    @discardableResult public mutating func removeValue(forKey key: Key) -> Value? {
        let index = self.index(forKey: key)

        // Find the element in the bucket's chain and remove it.
        for (i, element) in buckets[index].enumerated() {
            if element.key == key {
                buckets[index].remove(at: i)
                count -= 1
                return element.value
            }
        }
        return nil  // 不存在则返空
    }

    /**
     移除所有键值对
     */
    public mutating func removeAll() {
        buckets = Array<Bucket>(repeatElement([], count: buckets.count))
        count = 0
    }

    /**
     返回指定键的数组索引
     */
    private func index(forKey key: Key) -> Int {
        return abs(key.hashValue) % buckets.count
    }
}

/// 该扩展是选择性的，但是有更方便。
extension HashTable: CustomStringConvertible {
    
    public var description: String {
        let pairs = buckets.flatMap { b in b.map { e in "\(e.key) = \(e.value)" } }
        return pairs.joined(separator: ", ")
    }
    
    public var debugDescription: String {
        var str = ""
        for (i, bucket) in buckets.enumerated() {
            let pairs = bucket.map { e in "\(e.key) = \(e.value)" }
            str += "bucket \(i): " + pairs.joined(separator: ", ") + "\n"
        }
        return str
    }
}
```
引自 [Swift Algorithm club](https://github.com/raywenderlich/swift-algorithm-club) by [raywenderlich](https://www.raywenderlich.com)
### 1.3 实现一个栈

栈比较简单，只要对可变数组做些限制就可以了

```swift
struct Stack<E> {
    var elements = [E]()
    
    mutating func push(_ e: E) {
        elements.append(e)
    }
    
    mutating func pop() -> E? {
        return elements.popLast()
    }
}

extension Stack {
    var topElement: E? {
        return elements.last
    }
}

extension Stack: CustomStringConvertible {
    
    var description: String {
        return elements.description
    }
}
```
### 1.4 实现一个队列

通过数组实现的队列，重点在于如何优化解决出队时，数组后续元素整体迁移的问题，从而降低时间复杂度

```swift
public struct Queue<T> {
  fileprivate var array = [T?]()
  fileprivate var head = 0

  public var isEmpty: Bool {
    return count == 0
  }

  public var count: Int {
    return array.count - head
  }

  public mutating func enqueue(_ element: T) {
    array.append(element)
  }

  public mutating func dequeue() -> T? {
    guard head < array.count, let element = array[head] else { return nil }

    array[head] = nil
    head += 1

    let percentage = Double(head)/Double(array.count)
    if array.count > 50 && percentage > 0.25 {
      array.removeFirst(head)
      head = 0
    }

    return element
  }

  public var front: T? {
    if isEmpty {
      return nil
    } else {
      return array[head]
    }
  }
}
```
引自 [Swift Algorithm club](https://github.com/raywenderlich/swift-algorithm-club) by [raywenderlich](https://www.raywenderlich.com)
### 1.5 实现一个LRU缓存

```swift
import Foundation

public class LRUCache<KeyType: Hashable> {
  private let maxSize: Int
  private var cache: [KeyType: Any] = [:]
  private var priority: LinkedList<KeyType> = LinkedList<KeyType>()
  private var key2node: [KeyType: LinkedList<KeyType>.LinkedListNode<KeyType>] = [:]
  
  public init(_ maxSize: Int) {
    self.maxSize = maxSize
  }
  
  public func get(_ key: KeyType) -> Any? {
    guard let val = cache[key] else {
      return nil
    }
    
    remove(key)
    insert(key, val: val)
    
    return val
  }
  
  public func set(_ key: KeyType, val: Any) {
    if cache[key] != nil {
      remove(key)
    } else if priority.count >= maxSize, let keyToRemove = priority.last?.value {
      remove(keyToRemove)
    }
    
    insert(key, val: val)
  }
  
  private func remove(_ key: KeyType) {
    cache.removeValue(forKey: key)
    guard let node = key2node[key] else {
      return
    }
    priority.remove(node: node)
    key2node.removeValue(forKey: key)
  }
  
  private func insert(_ key: KeyType, val: Any) {
    cache[key] = val
    priority.insert(key, atIndex: 0)
    guard let first = priority.first else {
      return
    }
    key2node[key] = first
  }
}
```

引自 [Swift Algorithm club](https://github.com/raywenderlich/swift-algorithm-club) by [raywenderlich](https://www.raywenderlich.com)

## 2. 常用类型的数据结构

### 2.1 图片在内存的中的数据结构是什么样的？

二维数组，第一维度是每个元素代表一个像素，第二维度的每个元素代表一个像素的信息，通常是颜色空间，即色域，常见有的 YUV、RGB、RGBA 等。

## 3. 常见底层函数的实现

### 3.1 malloc 函数如何实现的

初始自己真实太天真，知道看见这个详解介绍。

```c
#define BLOCK_SIZE 24
void *first_block=NULL;
 
/* other functions... */
 
void *malloc(size_tsize){
    t_blockb,last;
    size_ts;
    /* 对齐地址 */
    s= align8(size);
    if(first_block){
        /* 查找合适的block */
        last= first_block;
        b= find_block(&last,s);
        if(b){
            /* 如果可以，则分裂 */
            if((b->size- s)>= (BLOCK_SIZE +8))
                split_block(b,s);
            b->free= 0;
        }else {
            /* 没有合适的block，开辟一个新的 */
            b= extend_heap(last,s);
            if(!b)
                returnNULL;
        }
    }else {
        b= extend_heap(NULL,s);
        if(!b)
            returnNULL;
        first_block= b;
    }
    returnb->data;
}

```


引自 [malloc 函数实现原理](https://blog.csdn.net/yeditaba/article/details/53443792)

#### 参考

[浅析malloc（）的几种实现方式](http://lionwq.spaces.eepw.com.cn/articles/article/item/18555)
[Anatomy of a Program in Memory](https://manybutfinite.com/post/anatomy-of-a-program-in-memory/)
[How The Kernel Manages Your Memory](https://manybutfinite.com/post/how-the-kernel-manages-your-memory/)
[GNU的malloc.c](https://repo.or.cz/w/glibc.git/blob/HEAD:/malloc/malloc.c)
