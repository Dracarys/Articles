# 面试题系列之数据库

## 1. 数据库基础

### 索引的作用、优缺点，与主键的区别

### 乐观锁VS悲观锁

## 2. SQLite

[深入理解SQLite](https://www.kancloud.cn/kangdandan/sqlite/64326)

### SQLite中插入特殊字符的方法和接受的处理方法？

```
public static String sqliteEscape(String keyWord){
    keyWord = keyWord.replace("/", "//");
    keyWord = keyWord.replace("'", "''");
    keyWord = keyWord.replace("[", "/[");
    keyWord = keyWord.replace("]", "/]");
    keyWord = keyWord.replace("%", "/%");
    keyWord = keyWord.replace("&","/&");
    keyWord = keyWord.replace("_", "/_");
    keyWord = keyWord.replace("(", "/(");
    keyWord = keyWord.replace(")", "/)");
    return keyWord;
}
```
### SQLite 与 MySQL 区别

## 3. CoreData

### CoreData的原理，与SQLite相比优劣？

### CoreData的6个成员类

### CoreData 多线程中处理大量数据同步时的操作？

### NSpersistentStoreCoordinator，NSManagedObjectContext 和 NSManagedObject 中的哪些需要在线程中创建或者传递？你用过什么样的策略实现的？


## 4. MySQL






## 5. 项目经验
### 如果不用数据库，只使用普通文件，如何设计亿量级别的日志系统？

### 数据库设计、字段设计等项目方案