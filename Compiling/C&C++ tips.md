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
