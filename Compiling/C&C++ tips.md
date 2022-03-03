# C&C++ tips

## __BEGIN_DECLS 和 __END_DECLS
这一对宏配合使用，其意义在于与C++混编时指明被宏所包裹的代码是C代码。宏定义如下：
```c
/* C++ needs to know that types and declarations are C, not C++.  */
#ifdef	__cplusplus
# define __BEGIN_DECLS	extern "C" {
# define __END_DECLS	}
#else
# define __BEGIN_DECLS
# define __END_DECLS
#endif
```

## Rang for staement
范围 for 语句。又掉这个坑里了，还是记录一下。

```c++
for (declaration : expression) 
    statement
```

expression 表示的必须是一个序列（实际上是能返回迭代器的 begin 和 end 成员）。declaration 则定义了一个变量，序列中的每个元素都得能转换成该变量的类型。既然涉及到转换，那么这里究竟是采用的那种转换方式呢 ——— 拷贝构造。所以 statement 中做的修改是不会影响到原有序列中的元素的。那么如果么有要修改的需要怎么办呢 ———— 引用。

